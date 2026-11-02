# RFC — Webhooks de saída para mudança de status de pedidos

> Legenda de rastreabilidade (herdada das ADRs): **[Reunião]** = discutido e decidido na reunião; **[Código]** = estado do repositório; **[Inferido]** = dedução dos autores das ADRs, **não** discutida na reunião. Referências `[ADR-00X]` apontam para a seção 8.

## 1. Metadados

| Campo | Valor |
|---|---|
| Título | Webhooks de saída para mudança de status de pedidos |
| Autor | Felipe Manfrin usando GPT 6 e Sonnet 5.5 |
| Status | Proposto para revisão |
| Data | 01/11/2026 (data de geração) |
| Revisores | Larissa, Diego, Bruno, Sofia, Marcos (participantes da reunião citados nas ADRs) |

A participação na reunião não representa aprovação formal deste RFC. Segundo [ADR-001], Sofia e Marcos já haviam saído da call quando duas decisões secundárias (UUID e snapshot do payload) foram tomadas, apenas com Larissa, Diego e Bruno.

## 2. Resumo executivo (TL;DR)

**Problema.** Três clientes B2B (Atlas Comercial, MaxDistribuição e Nova Cargo) consultam `GET /orders` periodicamente e pediram notificação quando o status de um pedido muda. "Tempo real" significa menos de 10 segundos. O escopo é apenas webhook de saída. [ADR-001]

**Abordagem.** Padrão *transactional outbox* no MySQL existente: o evento é gravado na mesma transação da mudança de status. Um worker em processo separado faz polling a cada 2 segundos e envia o evento por HTTP, assinado com HMAC-SHA256 com secret por endpoint. Falhas são reenviadas com backoff exponencial e, esgotadas as tentativas, vão para uma Dead Letter Queue (DLQ) com replay manual restrito a ADMIN. A garantia é *at-least-once*, e a deduplicação fica a cargo do cliente via `X-Event-Id`. Tudo reaproveita os padrões já existentes na codebase, sem infraestrutura nova.

**Decisões-chave.** Outbox transacional ([ADR-001]); worker separado, polling e processamento sequencial ([ADR-002]); retry de 5 tentativas e DLQ ([ADR-003]); HMAC-SHA256 com secret por endpoint e rotação com convivência de 24 h ([ADR-004]); at-least-once ([ADR-005]); reuso máximo de padrões existentes ([ADR-006]).

**Principais pendências.** Rate limiting por cliente; se a criação de pedido emite evento; ambiguidade na contagem de tentativas; semântica do replay; ordering com mais de um worker; controle de quem configura webhooks de qual cliente.

## 3. Contexto e problema

**Cenário atual.** Os clientes fazem polling em `GET /orders`. Não existe nenhum mecanismo de eventos, filas ou webhooks no repositório. [ADR-001, Código]

**Necessidade de negócio.** Notificar os clientes B2B de mudanças de status em menos de 10 segundos, via webhook de saída. [ADR-001]

**Restrições e requisitos identificados na reunião:**

- A transação de mudança de status já é pesada. Um cliente lento ou fora do ar não pode travar mudanças de outros pedidos, nem causar reversão da mudança. [ADR-001]
- O MySQL não oferece listener nativo equivalente ao NOTIFY/LISTEN do Postgres. [ADR-002]
- Os eventos contêm dados de pedidos e saem para endpoints fora da infraestrutura, então o cliente precisa validar origem e integridade. [ADR-004]
- Time pequeno: evitar infraestrutura nova e *overengineering*. [ADR-001, ADR-006]
- Clientes podem ter indisponibilidade de cerca de duas horas em manutenção planejada. [ADR-003]

## 4. Proposta técnica

**Componentes e responsabilidades**

1. **Publicação transacional do evento.** Na mesma transação SQL que atualiza o pedido e seu histórico, o sistema insere uma linha numa outbox. Se a inserção falhar, tudo sofre rollback, de modo que não existe mudança de status sem evento. O payload já renderizado (snapshot) é gravado nesse momento, para refletir o pedido como estava na mudança. O filtro por status de interesse do webhook é aplicado na inserção. [ADR-001]
2. **Worker separado.** Um novo processo Node, com seu próprio cliente Prisma e usando o mesmo banco, consulta a outbox a cada 2 segundos (pior caso de latência de polling aceito), processa os eventos pendentes mais antigos em ordem de criação e os envia por HTTP com timeout de 10 segundos. Timeout ou erro de entrega contam como falha. [ADR-002, ADR-003]
3. **Retry e DLQ.** Cada falha reagenda o evento com backoff exponencial (1 min, 5 min, 30 min, 2 h e 12 h). Esgotadas as tentativas, o evento vai para uma tabela de DLQ separada, preservada para debug. O replay é manual, feito por endpoint administrativo restrito à role ADMIN e deve registrar quem o executou. [ADR-003]
4. **Autenticação do envio.** Cada requisição é assinada com HMAC-SHA256 sobre o corpo, usando uma secret única por endpoint, gerada pela plataforma. A rotação é solicitada pelo cliente, e a secret antiga permanece válida por 24 horas. Além da assinatura, a requisição leva cabeçalhos com o id do evento, o timestamp do envio e o id do endpoint. [ADR-004]
5. **Garantia de entrega.** *At-least-once*: o cliente pode receber o mesmo evento mais de uma vez e deduplica usando o identificador do evento. O comportamento será documentado de forma destacada no portal de desenvolvedores. [ADR-005]
6. **Gestão de webhooks.** API sob o JWT do sistema, aberta a qualquer role autenticada "por enquanto", com possibilidade de endurecer depois. [ADR-004]
7. **Organização do código.** Novo módulo de webhooks seguindo o padrão por domínio, reutilizando logger, tratamento de erros, validação, autenticação, Prisma e UUID existentes. [ADR-006]

**Limitação documentada.** Há ordering por pedido apenas enquanto houver um único worker. Isso não é garantia de ordering global, e os clientes nunca pediram essa garantia. [ADR-002]

**Fora de escopo (decidido na reunião):** arquivamento de eventos entregues (cerca de 30 dias) e aviso por e-mail ao cliente em caso de falhas. [ADR-001, ADR-003]

Detalhes de implementação (tabelas, colunas, assinaturas, rotas, formatos) pertencem ao FDD.

## 5. Alternativas consideradas

Todas as alternativas abaixo foram discutidas na reunião. Alternativas marcadas como `[Inferido]` nas ADRs (por exemplo CDC pelo binlog, mTLS/OAuth2 ou circuit breaker) não foram incluídas aqui.

| Alternativa | Por que foi considerada | Trade-off que levou à rejeição | ADR |
|---|---|---|---|
| **Chamada HTTP síncrona na transação de mudança de status** | Caminho mais direto para notificar | Acopla a transação à disponibilidade do cliente. Um cliente lento ou fora do ar travaria outras mudanças, e reverter o status por falha do cliente não faz sentido. | [ADR-001] |
| **Redis Streams ou similar** | Mecanismo de fila/evento | Exigiria infraestrutura nova. Foi considerado *overengineering* para um time pequeno. | [ADR-001] |
| **Trigger no banco ou LISTEN/NOTIFY para acionar o worker** | Evitar polling | Triggers só executam SQL e não avisam processos externos. LISTEN/NOTIFY é inviável no MySQL. | [ADR-002] |
| **Worker dentro do processo da API** | Simplicidade de deploy | Um restart da API derrubaria o worker. | [ADR-002] |
| **Exactly-once** | Evitar duplicatas | Exigiria coordenação dos dois lados e seria muito mais complexo. | [ADR-005] |
| **Secret global da plataforma** | Gestão mais simples | O vazamento de uma secret comprometeria todos os clientes. | [ADR-004] |
| **3 tentativas de retry** | Política mais curta | Cobriria cerca de 30 minutos e perderia eventos em indisponibilidades maiores. | [ADR-003] |
| **Marcar `failed` na própria outbox em vez de DLQ separada** | Menos estruturas | A DLQ separada é mais limpa e serve de evidência para debug e reprocessamento. | [ADR-003] |

Também foram descartadas: guardar apenas `order_id` e renderizar o payload no envio (produziria "caso esquisito" se o pedido mudasse depois, [ADR-001]); retry indefinido ([ADR-003]); replay aberto a qualquer role ([ADR-003]).

## 6. Questões em aberto

### 6.1 Adiadas ou não decididas na reunião

1. **Rate limiting de envio por cliente.** Diego levantou o caso de 50 pedidos por minuto, e a decisão foi "observar e decidir depois". [ADR-002]
2. **Múltiplos workers e ordering.** Escalar para vários workers (particionando por `order_id` ou com lock pessimista) foi adiado como "problema do futuro". Com vários workers, a ordenação por pedido deixa de ser garantida. [ADR-002]
3. **Aviso por e-mail ao cliente em caso de falhas.** Adiado, talvez para uma fase futura. [ADR-003]
4. **Arquivamento de eventos entregues.** Fora do escopo desta feature, sem previsão. [ADR-001]
5. **Endurecimento do acesso à API de gestão.** Qualquer role autenticada pode gerenciar webhooks "por enquanto". [ADR-004]
6. **Contagem de tentativas.** "5 tentativas" pode significar 5 envios no total ou 1 envio mais 5 retentativas. A soma dos intervalos (14h36) sugere 6 envios, mas Larissa falou em "total 5 tentativas". A reunião não resolveu isso. Diego também citou janelas de "12 ou 24 horas" e depois "quase 15". [ADR-003]
7. **Semântica do estado `falhou`.** Não está claro se é transitório (aguardando retry) ou equivale à falha permanente da DLQ. [ADR-003]
8. **Criação de pedido.** A reunião tratou apenas mudança de status, então não está decidido se a criação de pedido emite evento. [ADR-001]

### 6.2 Pontos identificados na análise das ADRs (não discutidos na reunião)

- **Cardinalidade evento × endpoint** e se o `event_id` é o mesmo identificador da linha da outbox ([ADR-001], [ADR-005]).
- **Replay:** se zera o contador de tentativas, se mantém o `event_id` original e se remove a linha da DLQ ([ADR-003], [ADR-005]).
- **Critério de falha:** quais códigos HTTP contam como falha; uso de jitter ([ADR-003]).
- **Segurança:** formato de `X-Signature`; como assinar durante o período de 24 h com duas secrets válidas; armazenamento da secret e se pode ser consultada após a criação ([ADR-004]).
- **Divergência código × reunião:** Marcos mencionou usuários que representam o cliente, mas no código `User` não tem vínculo com `Customer`. Com `customer_id` no body e CRUD aberto, não há controle sobre quem configura webhooks de qual cliente ([ADR-004]).
- **Operação do worker:** recuperação de linhas presas em "processando", tamanho do lote, shutdown gracioso, tratamento de erros e logs fora do pipeline do Express ([ADR-002], [ADR-006]).
- **Outbox e exclusão de pedidos:** comportamento esperado para eventos de pedidos que podem ser apagados ([ADR-001]).
- **Auditoria do replay:** log ou registro persistido; histórico de tentativas de entrega, sem local de armazenamento definido ([ADR-003]).
- **Convenções:** prefixo `/api/v1` nas rotas; códigos de erro `WEBHOOK_*`; nomenclatura das colunas ([ADR-001], [ADR-003], [ADR-006]).

## 7. Impactos e riscos

### 7.1 Consequências discutidas na reunião

- **Confiabilidade:** atomicidade entre pedido e evento, sem inconsistência quando o commit ocorre. [ADR-001]
- **Infraestrutura:** nenhuma infraestrutura nova além de um processo Node. [ADR-002, ADR-006]
- **Desempenho:** latência de polling de até 2 segundos, aceita pelo PM. O filtro na inserção economiza linhas. [ADR-001, ADR-002]
- **Limitações conhecidas:** ordering só por pedido e só com um worker. Cliente indisponível por cerca de 15 horas terá "problema sério" (aceito por Marcos). Reprocessamento depende de ação humana com perfil ADMIN. [ADR-002, ADR-003]
- **Responsabilidade do cliente:** deduplicar eventos (Sofia: "joga responsabilidade pro cliente"). [ADR-005]
- **Segurança:** vazamento de uma secret afeta um único endpoint; a rotação não exige parada do cliente. O código de segurança (HMAC e geração de secret) exige pelo menos dois dias úteis de revisão por Sofia antes do deploy. [ADR-004]
- **Consistência:** reuso de padrões reduz código novo. [ADR-006]

### 7.2 Riscos e impactos inferidos (não discutidos na reunião)

- **Desempenho / meta de latência:** com processamento sequencial e timeout de 10 s, um endpoint lento ou fora do ar pode atrasar eventos de todos os clientes (*head-of-line blocking*) e comprometer a meta de menos de 10 s. [ADR-002]
- **Complexidade transacional:** a transação de mudança de status ganha ao menos mais um `INSERT`, numa transação que Bruno já considera pesada. A outbox cresce no mesmo MySQL dos dados de negócio, sem arquivamento previsto. [ADR-001]
- **Ordering e duplicatas:** um evento em espera de retry pode ser ultrapassado por outro do mesmo pedido, e o cliente pode receber eventos duplicados e fora de ordem. [ADR-002, ADR-005]
- **Disponibilidade:** com o worker parado, os eventos acumulam sem entrega, pois não há alta disponibilidade. [ADR-002]
- **Alcance da garantia:** at-least-once vale dentro da política de retry. Eventos na DLQ só são entregues após replay. [ADR-003, ADR-005]
- **Segurança:** o `X-Timestamp` não é coberto pela assinatura, o que limita a detecção de replay pelo cliente. A secret precisa ser legível pelo worker. A redação de logs atual não cobre secret ou assinatura, e já houve cliente que vazou secret em log. [ADR-004]
- **Acoplamento:** o módulo de pedidos passa a depender do módulo de webhooks. O registro do módulo exige mudanças na composição da aplicação e nas rotas. [ADR-006]
- **Reuso com ressalvas:** o middleware de erro cobre apenas rotas HTTP, e algumas classes de erro existentes não aceitam código customizado. [ADR-006]

Estes riscos são apontamentos para revisão, não requisitos nem decisões aprovadas.

## 8. Decisões relacionadas

Todas as ADRs referenciadas pelas fontes estavam disponíveis no pacote. Não há referências a ADRs ausentes.

| ID | Título | Arquivo |
|---|---|---|
| ADR-001 | Transactional Outbox com MySQL | [ADR-001-transactional-outbox-com-mysql.md](./ADR-001-transactional-outbox-com-mysql.md) |
| ADR-002 | Worker separado, polling e processamento sequencial | [ADR-002-worker-separado-polling-e-processamento-sequencial.md](./ADR-002-worker-separado-polling-e-processamento-sequencial.md) |
| ADR-003 | Política de retry e Dead Letter Queue (DLQ) | [ADR-003-politica-de-retry-e-dead-letter-queue.md](./ADR-003-politica-de-retry-e-dead-letter-queue.md) |
| ADR-004 | Autenticação e assinatura HMAC-SHA256 | [ADR-004-autenticacao-e-assinatura-hmac-sha256.md](./ADR-004-autenticacao-e-assinatura-hmac-sha256.md) |
| ADR-005 | Garantia de entrega at-least-once e deduplicação por X-Event-Id | [ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md](./ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md) |
| ADR-006 | Reutilização dos padrões arquiteturais existentes | [ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md](./ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md) |