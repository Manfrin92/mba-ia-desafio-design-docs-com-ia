# PRD — Sistema de Webhooks de Notificação de Pedidos

**Produto:** Order Management System (OMS)  
**Feature:** Webhooks de saída para mudança de status de pedidos  
**Status:** Documento de requisitos consolidado a partir de documentos propostos para revisão e ADRs aceitas  
**Data:** 02/11/2026

> **Convenção de leitura:** decisões arquiteturais citadas neste documento estão registradas nas ADRs com status **Aceito**. Pontos ainda não decididos são identificados como pendências. Metas sugeridas são explicitamente diferenciadas de compromissos aprovados.

## 1. Resumo e contexto da feature

O Sistema de Webhooks de Notificação de Pedidos permitirá que clientes B2B do OMS recebam notificações HTTP quando ocorrer uma mudança de status em pedidos de seu interesse.

Atualmente, clientes como Atlas Comercial, MaxDistribuição e Nova Cargo consultam periodicamente a API de pedidos para identificar alterações. A nova feature oferece uma forma de comunicação orientada a eventos, permitindo que esses clientes reajam às mudanças sem depender de consultas recorrentes.

A solução contempla o cadastro e a gestão de endpoints de notificação, a seleção dos status de interesse, o envio de eventos, a consulta do histórico de entregas e mecanismos para lidar com falhas e reprocessar notificações que não puderam ser entregues automaticamente.

O escopo desta entrega é exclusivamente de **webhooks de saída**: o OMS envia notificações para sistemas externos. A necessidade de negócio é que essas notificações sejam recebidas em menos de 10 segundos em condições normais de operação.

## 2. Problema e motivação

Os clientes B2B precisam conhecer as mudanças de status dos pedidos para atualizar seus próprios sistemas e dar continuidade aos processos de negócio. Hoje, precisam consultar repetidamente `GET /orders` para descobrir se houve alguma alteração.

Esse modelo apresenta limitações:

- O cliente depende de consultas periódicas para identificar mudanças.
- A atualização de seus sistemas não ocorre de forma orientada a eventos e pode sofrer atraso entre as consultas.
- As consultas recorrentes geram tráfego e trabalho desnecessários para os sistemas envolvidos.
- Não há, atualmente, um mecanismo de notificação externa no OMS.

A necessidade foi explicitada por três clientes B2B. Para eles, “tempo real” significa receber a notificação em menos de 10 segundos. A Atlas Comercial também condicionou sua permanência à entrega da capacidade até o fim do trimestre.

A feature busca reduzir a dependência de polling, facilitar a integração entre o OMS e os sistemas dos clientes e oferecer mecanismos de confiabilidade, segurança e acompanhamento das notificações.

## 3. Público-alvo e cenários de uso

### 3.1. Público-alvo

- **Clientes B2B integradores:** empresas que mantêm sistemas próprios e precisam refletir mudanças de status dos pedidos recebidos pelo OMS.
- **Equipes técnicas dos clientes:** responsáveis por configurar endpoints, validar notificações recebidas e tratar duplicidades.
- **Operadores e administradores do OMS:** usuários autenticados que gerenciam configurações de webhook; administradores também podem reprocessar eventos que chegaram à fila de falhas permanentes.
- **Equipe responsável pelo produto e pela operação do OMS:** beneficiada pela redução da dependência de consultas recorrentes e pela capacidade de acompanhar falhas de entrega.

### 3.2. Cenários de uso

**Cenário 1 — Receber mudanças de status**

Um cliente B2B deseja atualizar seu sistema quando um pedido passa de `PROCESSING` para `SHIPPED`. Ele cadastra um endpoint e seleciona os status de interesse. Quando uma mudança correspondente ocorre no OMS, o sistema envia a notificação ao endpoint configurado.

**Cenário 2 — Gerenciar inscrições**

Um usuário autenticado consulta os webhooks de um cliente, altera a URL de destino, modifica os status de interesse, ativa ou desativa uma configuração, ou remove um endpoint.

**Cenário 3 — Validar a origem da notificação**

O sistema do cliente recebe uma notificação e verifica sua assinatura criptográfica para validar a origem e a integridade do corpo recebido.

**Cenário 4 — Renovar credenciais de integração**

Um cliente solicita a rotação da credencial de um endpoint. A credencial anterior permanece válida por 24 horas, permitindo que o cliente atualize sua integração sem interrupção imediata.

**Cenário 5 — Investigar entregas**

Uma equipe técnica consulta as últimas 100 tentativas de entrega de um endpoint, incluindo informações sobre o resultado, a resposta recebida e o tempo de resposta, para investigar problemas de integração.

**Cenário 6 — Recuperar notificações não entregues**

Quando as tentativas automáticas são esgotadas, um administrador consulta a situação da falha e solicita manualmente o reprocessamento do evento, com registro de quem realizou a operação.

## 4. Objetivos e métricas de sucesso

| Objetivo | Indicador | Meta / referência | Natureza |
|---|---|---|---|
| Reduzir o atraso na comunicação de mudanças de status | Tempo entre a mudança do pedido e a entrega da notificação | Menos de 10 segundos | Requisito definido na reunião e nas ADRs |
| Validar o desempenho da entrega em condições controladas | Percentil 95 do tempo de entrega em homologação, para 100 eventos consecutivos e endpoint saudável | p95 inferior a 10 segundos | Critério de validação registrado no FDD |
| Garantir que mudanças de status não fiquem sem o evento correspondente | Consistência entre mudança de status confirmada e evento registrado | Nenhuma mudança confirmada sem o evento correspondente quando houver endpoint interessado | Decisão de atomicidade da ADR-001 |
| Permitir recuperação de falhas de entrega | Eventos que esgotam as tentativas ficam disponíveis para investigação e replay administrativo | 100% dos eventos que esgotarem a política devem ser encaminhados à DLQ | Comportamento definido nas ADRs |
| Permitir que clientes validem a origem das notificações | Requisições entregues com assinatura HMAC-SHA256 e credencial específica do endpoint | Assinatura presente em todos os envios | Decisão da ADR-004 |

**Observação sobre as metas:** a referência de menos de 10 segundos é uma necessidade de negócio registrada. O FDD estabelece um cenário de homologação com 100 eventos consecutivos e p95 inferior a 10 segundos. Isso não representa uma garantia absoluta para situações de indisponibilidade, backlog ou endpoint lento. A meta de 100% na DLQ expressa a cobertura esperada do fluxo definido, não uma métrica histórica já observada.

A RFC e as ADRs não apresentam uma linha de base quantitativa de polling, volume de eventos, taxa atual de falhas ou disponibilidade. Esses indicadores deverão ser coletados para permitir avaliações posteriores de impacto.

## 5. Escopo

### 5.1. Dentro do escopo

- Notificações de saída para mudanças de status de pedidos.
- Cadastro de endpoints HTTPS associados a clientes.
- Seleção dos status de pedido que cada endpoint deseja receber.
- Consulta, atualização, ativação, desativação e remoção de configurações de webhook.
- Geração de uma credencial exclusiva por endpoint e possibilidade de rotação, com convivência de 24 horas entre a credencial anterior e a nova.
- Envio de notificações com assinatura HMAC-SHA256.
- Identificação dos eventos por um identificador que permita ao consumidor reconhecer notificações repetidas.
- Registro e consulta do histórico das últimas 100 tentativas de entrega por endpoint.
- Reenvio automático após falhas, segundo a política de tentativas e intervalos definida.
- Encaminhamento de eventos com falha permanente para uma DLQ.
- Reprocessamento manual de eventos da DLQ por usuários com perfil ADMIN, com registro de autoria da operação.
- Documentação do comportamento de entrega e das responsabilidades de integração para os clientes.

### 5.2. Fora de escopo

**Itens explicitamente descartados ou excluídos desta entrega**

| Item | Classificação | Evidência |
|---|---|---|
| Webhooks de entrada, nos quais clientes enviam eventos para o OMS | Descartado para esta feature | ADR-001 |
| Arquivamento de eventos entregues após aproximadamente 30 dias | Excluído do escopo, sem previsão definida | ADR-001 |
| Deduplicação de eventos no servidor e garantia *exactly-once* | Descartado em favor de *at-least-once* e deduplicação pelo consumidor | ADR-005 |
| Chamada HTTP síncrona durante a transação de mudança de status | Alternativa rejeitada | ADR-001 |

**Itens adiados ou deixados para avaliação futura**

| Item | Classificação | Evidência |
|---|---|---|
| Aviso por e-mail ao cliente em caso de falhas | Adiado para possível fase futura | ADR-003 |
| Rate limiting de envio por cliente | Adiado; a decisão foi observar o comportamento antes de definir limites | ADR-002 |
| Execução com múltiplos workers em paralelo | Adiado para uma necessidade futura de escala | ADR-002 |
| Endurecimento das permissões da API de gestão para restringir usuários ao próprio cliente | Pendente de decisão; a configuração atual permite qualquer role autenticada e não estabelece vínculo entre usuário e cliente | ADR-004 |

Também não está decidido se a criação de um pedido deve gerar uma notificação. Portanto, esta entrega considera apenas mudanças de status explicitamente contempladas nas fontes, sem assumir a emissão de evento na criação.

## 6. Requisitos funcionais

Os requisitos abaixo descrevem capacidades observáveis do produto. Os identificadores são próprios deste PRD e não representam novos compromissos além dos comportamentos fundamentados nas fontes.

### 6.1. Gestão de endpoints

| ID | Requisito |
|---|---|
| **PRD-FR-01** | O sistema deve permitir que um usuário autenticado cadastre um endpoint de webhook associado a um cliente, informando a URL de destino e os status de pedido que deseja receber. |
| **PRD-FR-02** | O sistema deve permitir consultar os endpoints cadastrados para um cliente, sem expor as credenciais secretas desses endpoints nas respostas de consulta. |
| **PRD-FR-03** | O sistema deve permitir atualizar a URL, a descrição, a situação de ativação e os status de interesse de um endpoint. As alterações devem valer para eventos futuros, sem modificar o conteúdo dos eventos já registrados. |
| **PRD-FR-04** | O sistema deve permitir desativar e remover um endpoint, impedindo que novas notificações sejam destinadas a uma configuração inativa ou removida, conforme o comportamento de remoção definido para a feature. |
| **PRD-FR-05** | O sistema deve permitir que cada endpoint selecione os status de pedido para os quais deseja receber notificações. Eventos de status não selecionados não devem gerar notificações para aquele endpoint. |

### 6.2. Notificações e entrega

| ID | Requisito |
|---|---|
| **PRD-FR-06** | O sistema deve gerar uma notificação quando o status de um pedido for alterado para um status de interesse de pelo menos um endpoint ativo do cliente correspondente. |
| **PRD-FR-07** | O sistema deve preservar na notificação os dados do pedido e os status anterior e novo correspondentes ao momento em que a mudança ocorreu, mesmo que o pedido seja alterado posteriormente. |
| **PRD-FR-08** | O sistema deve encaminhar as notificações aos endpoints externos de forma assíncrona, sem depender da disponibilidade ou do tempo de resposta desses endpoints para confirmar a mudança de status do pedido. |
| **PRD-FR-09** | O sistema deve tentar novamente a entrega de notificações que falharem, respeitando a política de backoff e o limite de tentativas definidos para a feature. |
| **PRD-FR-10** | O sistema deve encaminhar notificações cujas tentativas tenham sido esgotadas para uma área de falhas permanentes, preservando informações que permitam investigação e recuperação. |
| **PRD-FR-11** | O sistema deve permitir que um usuário com perfil ADMIN solicite o reprocessamento manual de uma notificação na DLQ, registrando quem executou a ação. |
| **PRD-FR-12** | O sistema deve permitir consultar as últimas 100 tentativas de entrega de um endpoint, incluindo informações suficientes para identificar o evento, o resultado, a resposta recebida e o tempo de resposta. |

### 6.3. Segurança e integração

| ID | Requisito |
|---|---|
| **PRD-FR-13** | O sistema deve assinar as notificações enviadas com HMAC-SHA256, utilizando uma credencial exclusiva para cada endpoint, para que o consumidor possa verificar a origem e a integridade do corpo recebido. |
| **PRD-FR-14** | O sistema deve permitir a rotação da credencial de um endpoint, mantendo a credencial anterior válida por 24 horas após a rotação. A nova credencial deve ser disponibilizada ao solicitante no momento da operação. |
| **PRD-FR-15** | O sistema deve identificar cada evento com um identificador único disponibilizado ao consumidor, permitindo que este reconheça e deduplique notificações recebidas mais de uma vez. |

## 7. Requisitos não funcionais

| ID | Característica | Requisito / restrição |
|---|---|---|
| **PRD-NFR-01** | Desempenho | A necessidade de negócio é que notificações sejam entregues em menos de 10 segundos. O FDD estabelece p95 inferior a 10 segundos em homologação, com 100 eventos consecutivos e endpoint saudável. Não há garantia documentada para situações de backlog ou endpoint lento. |
| **PRD-NFR-02** | Confiabilidade | A mudança de status e o registro do evento correspondente devem ser atômicos. Uma falha ao registrar o evento deve impedir que a mudança de status seja confirmada sem ele. |
| **PRD-NFR-03** | Confiabilidade | A entrega segue semântica *at-least-once*. O consumidor deve estar preparado para receber o mesmo evento mais de uma vez e deduplicá-lo pelo identificador disponibilizado. |
| **PRD-NFR-04** | Resiliência | Falhas de entrega devem ser tratadas por tentativas automáticas limitadas e, após o esgotamento, por encaminhamento à DLQ e possibilidade de replay manual. A cobertura não é indefinida: eventos na DLQ dependem de ação administrativa. |
| **PRD-NFR-05** | Segurança | Os endpoints de destino devem utilizar HTTPS. As notificações devem utilizar assinatura HMAC-SHA256 com credencial exclusiva por endpoint. |
| **PRD-NFR-06** | Segurança | O corpo da notificação deve respeitar o limite de 64 KB definido no FDD. Payloads que excedam esse limite devem ser rejeitados conforme a validação prevista. |
| **PRD-NFR-07** | Segurança | A credencial de cada endpoint deve poder ser rotacionada com período de convivência de 24 horas entre a credencial anterior e a nova. |
| **PRD-NFR-08** | Segurança e governança | O código relacionado à geração e ao uso das credenciais e à assinatura HMAC deve passar por pelo menos dois dias úteis de revisão de segurança antes do deploy, conforme condição registrada na ADR-004. |
| **PRD-NFR-09** | Observabilidade | Deve ser possível acompanhar tentativas de entrega, seus resultados e tempos de resposta, consultar o histórico recente e investigar falhas permanentes. O FDD também prevê logs estruturados e correlação entre API, eventos e entregas. |
| **PRD-NFR-10** | Disponibilidade e operação | O processamento de notificações deve ser independente do processo da API. A decisão reduz a dependência entre os ciclos de vida dos processos, mas não estabelece alta disponibilidade do worker; sua indisponibilidade pode acumular eventos sem entregá-los. |
| **PRD-NFR-11** | Escalabilidade | A primeira versão opera com um único worker e processamento sequencial. A execução paralela de múltiplos workers e o rate limiting por cliente não fazem parte das capacidades comprometidas nesta entrega. |

Não foram encontradas nas fontes metas quantitativas aprovadas para disponibilidade, throughput, volume máximo de eventos, tempo de recuperação do worker ou retenção do histórico de entregas. Esses valores não são presumidos neste PRD.

## 8. Decisões e trade-offs principais

As decisões abaixo estão registradas nas seis ADRs com status **Aceito**. Os impactos são apresentados sob a perspectiva do produto, sem reproduzir o desenho de implementação.

| ADR | Decisão consolidada | Benefício para o produto | Limitação / trade-off |
|---|---|---|---|
| [ADR-001 — Transactional Outbox com MySQL](./adrs/ADR-001-transactional-outbox-com-mysql.md) | Registrar o evento de forma atômica com a mudança de status, utilizando o banco existente. | Evita confirmar uma mudança de status sem registrar o evento correspondente e dispensa infraestrutura adicional de mensageria. | Acrescenta trabalho à transação de mudança de status. O arquivamento de eventos entregues foi excluído do escopo. |
| [ADR-002 — Worker separado, polling e processamento sequencial](./adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md) | Processar notificações em processo separado, consultando eventos periodicamente, com um único worker. | Isola o processamento da API e permite continuar tentando entregas mesmo quando a API é reiniciada. O intervalo de polling de 2 segundos foi aceito para atender à necessidade de latência. | Não há alta disponibilidade do worker. O processamento sequencial pode fazer um endpoint lento atrasar notificações de outros clientes. A ordenação por pedido é limitada ao cenário de um único worker. |
| [ADR-003 — Política de retry e Dead Letter Queue](./adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md) | Realizar tentativas limitadas com backoff exponencial e encaminhar falhas permanentes para DLQ, com replay manual por ADMIN. | Aumenta a tolerância a indisponibilidades temporárias e preserva eventos que precisam de investigação. | A cobertura é limitada à janela de tentativas definida. A recuperação de eventos na DLQ depende de intervenção administrativa. A contagem exata de tentativas ainda apresenta ambiguidade na reunião. |
| [ADR-004 — Autenticação e assinatura HMAC-SHA256](./adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md) | Assinar o corpo das notificações com credencial exclusiva por endpoint e permitir rotação com convivência de 24 horas. | Permite que o consumidor valide a origem e a integridade da notificação. O vazamento de uma credencial fica limitado ao endpoint correspondente. | A gestão de credenciais exige cuidados operacionais. A política de autorização da API de gestão permanece permissiva: qualquer role autenticada pode gerenciar webhooks, sem vínculo entre usuário e cliente definido no modelo atual. |
| [ADR-005 — Garantia at-least-once e deduplicação por X-Event-Id](./adrs/ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md) | Adotar entrega *at-least-once* e atribuir ao cliente a deduplicação pelo identificador do evento. | Evita a complexidade de coordenar uma garantia *exactly-once* entre OMS e sistemas externos. | O consumidor precisa implementar deduplicação e tolerar duplicatas. A ordenação de eventos não é garantida em todos os cenários. |
| [ADR-006 — Reutilização dos padrões arquiteturais existentes](./adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md) | Reutilizar os padrões, bibliotecas e estrutura modular já existentes no OMS. | Reduz a introdução de novas tecnologias e favorece consistência com o restante do produto. | A solução permanece condicionada às capacidades e limitações dos padrões atuais e exige integração do novo módulo com o fluxo existente de pedidos. |

## 9. Dependências

### 9.1. Dependências já existentes

- **OMS e ciclo de vida de pedidos:** o sistema já possui mudança de status controlada e histórico de alterações. A feature depende desse fluxo para identificar as transições relevantes.
- **MySQL e persistência existente:** o banco atual será utilizado para manter os registros necessários à publicação e ao processamento dos eventos.
- **Autenticação e autorização:** a API existente utiliza JWT e possui roles `ADMIN` e `OPERATOR`. A gestão de webhooks seguirá a decisão atual de permitir qualquer role autenticada; o replay exige ADMIN.
- **Padrões de aplicação:** estrutura modular, validação, tratamento de erros e logging existentes serão reutilizados, conforme ADR-006.
- **Clientes e endpoints externos:** a entrega depende de os clientes disponibilizarem endpoints HTTPS acessíveis e capazes de receber e validar as notificações.

### 9.2. Dependências a disponibilizar ou implementar

- **Novo módulo de webhooks e capacidades de gestão:** cadastro, consulta, atualização, remoção, rotação de credenciais e consulta de entregas ainda precisam ser implementados.
- **Processo de entrega independente:** o worker de notificações é um novo processo operacional e precisa ser disponibilizado e monitorado separadamente da API.
- **Fluxo de retry e DLQ:** os mecanismos de tentativas, armazenamento de falhas permanentes e replay administrativo precisam ser implementados.
- **Documentação para integradores:** o comportamento *at-least-once*, o identificador de deduplicação e a validação da assinatura precisam ser documentados no portal de desenvolvedores. A RFC atribui a redação final a Marcos.
- **Revisão de segurança:** a revisão do código de HMAC e geração de credenciais por Sofia, com antecedência mínima de dois dias úteis ao deploy, é condição explícita da ADR-004.

### 9.3. Dependências e definições pendentes

A autorização por cliente ainda não está resolvida: o modelo atual de usuário não estabelece vínculo com `Customer`, enquanto a gestão de webhooks recebe o identificador do cliente na requisição. Também precisam ser esclarecidas a semântica do replay, a contagem das tentativas e a emissão de eventos na criação de pedidos. Esses pontos não devem ser tratados como capacidades já aprovadas.

## 10. Riscos e mitigação

As probabilidades abaixo são avaliações qualitativas para planejamento do produto, não classificações aprovadas nas ADRs. Quando o risco deriva de uma consequência inferida, isso é indicado.

| Risco | Probabilidade | Impacto | Estratégia de mitigação |
|---|---|---|---|
| **Atraso de notificações causado por endpoint lento ou indisponível.** O processamento sequencial pode fazer um cliente atrasar entregas destinadas a outros clientes. Risco derivado da ADR-002 e explicitado como inferência na RFC. | Alta | Alto | Monitorar tempos de entrega e backlog; validar a meta de latência com endpoints saudáveis; acompanhar o impacto de endpoints lentos. Reavaliar paralelismo ou isolamento por cliente se as medições demonstrarem necessidade, respeitando que múltiplos workers foram adiados. |
| **Eventos não entregues automaticamente após indisponibilidade prolongada do cliente.** Após o limite de tentativas, a notificação vai para a DLQ e depende de replay administrativo. | Média | Alto | Monitorar falhas permanentes, disponibilizar consulta e replay restrito a ADMIN, manter evidências para investigação e documentar a janela limitada de tentativas para os clientes. |
| **Acesso indevido à configuração de webhooks de outro cliente.** A API aceita qualquer role autenticada e o modelo atual não relaciona usuário a cliente. Risco identificado nas ADR-004 e RFC. | Alta | Alto | Tratar a restrição por cliente como pendência de segurança prioritária; definir e aprovar a associação entre identidade e cliente antes de endurecer a autorização. Enquanto não houver decisão, explicitar a limitação e avaliar sua aceitação formal antes da disponibilização. |
| **Exposição ou uso indevido de credenciais de assinatura.** Há registro de vazamento de secrets em logs de clientes, e a RFC aponta que a redação atual dos logs não cobre todos os campos sensíveis. | Média | Alto | Revisar o tratamento de secrets e logs; garantir que credenciais não sejam expostas em consultas ou registros operacionais; cumprir a revisão de segurança mínima de dois dias úteis e orientar clientes sobre armazenamento seguro. |
| **Duplicidade ou recebimento fora de ordem de notificações.** A garantia *at-least-once* admite duplicatas, e a ordenação possui limitações em cenários de retry. | Alta | Médio | Documentar claramente a semântica de entrega; orientar clientes a deduplicar pelo identificador do evento e a não assumir ordenação global; validar os cenários de retry e replay. |
| **Acúmulo de eventos quando o worker fica indisponível.** Não foi definida alta disponibilidade do worker. Risco derivado da ADR-002. | Média | Alto | Monitorar a operação do worker e o acúmulo de eventos; estabelecer procedimentos operacionais para detectar e recuperar interrupções. A disponibilidade-alvo e o tempo de recuperação ainda precisam ser definidos. |

## 11. Critérios de aceitação

A feature será considerada aderente aos requisitos de produto quando os resultados observáveis abaixo forem demonstrados em homologação e nas revisões aplicáveis.

| ID | Critério de aceitação | Requisitos relacionados |
|---|---|---|
| **PRD-AC-01** | Um usuário autenticado consegue cadastrar um endpoint HTTPS para um cliente, escolher status de interesse e receber a credencial gerada no cadastro. A credencial não é exposta em consultas posteriores. | FR-01, FR-13 |
| **PRD-AC-02** | O usuário consegue consultar, atualizar e remover configurações de webhook. Alterações nos status de interesse afetam somente eventos futuros. | FR-02, FR-03, FR-04, FR-05 |
| **PRD-AC-03** | Quando um pedido muda para um status selecionado por um endpoint ativo, o cliente recebe uma notificação correspondente à mudança. | FR-06 |
| **PRD-AC-04** | A notificação representa os dados do pedido e os status no momento da mudança, sem ser alterada por mudanças posteriores no pedido. | FR-07 |
| **PRD-AC-05** | A indisponibilidade ou lentidão do endpoint externo não impede a confirmação da mudança de status do pedido no OMS. | FR-08, NFR-02 |
| **PRD-AC-06** | Com endpoint saudável e sem backlog, 100 eventos consecutivos em homologação apresentam p95 de entrega inferior a 10 segundos. | NFR-01 |
| **PRD-AC-07** | Uma falha de entrega provoca novas tentativas conforme a política definida; após o esgotamento, o evento fica disponível na DLQ para investigação. | FR-09, FR-10 |
| **PRD-AC-08** | Um usuário sem perfil ADMIN não consegue executar replay de evento da DLQ; um ADMIN consegue solicitar o replay e a ação fica associada ao usuário que a executou. | FR-11 |
| **PRD-AC-09** | O cliente consegue consultar as últimas 100 tentativas de entrega do endpoint e identificar evento, resultado, resposta e tempo de resposta. | FR-12 |
| **PRD-AC-10** | As notificações são assinadas com HMAC-SHA256 e utilizam credencial exclusiva do endpoint correspondente. | FR-13, NFR-05 |
| **PRD-AC-11** | Após a rotação de credencial, a anterior permanece válida durante 24 horas, e a nova credencial é disponibilizada ao solicitante. | FR-14, NFR-07 |
| **PRD-AC-12** | O consumidor recebe um identificador de evento estável para reconhecer notificações repetidas, e a documentação pública explica que duplicatas podem ocorrer e devem ser deduplicadas pelo cliente. | FR-15, NFR-03 |
| **PRD-AC-13** | A revisão de segurança do código de geração de credenciais e assinatura HMAC foi concluída com antecedência mínima de dois dias úteis antes do deploy. | NFR-08 |

Os critérios de aceitação não eliminam as pendências de produto e arquitetura registradas na RFC. Em particular, a autorização por cliente, a interpretação exata do limite de tentativas e a semântica do replay precisam ser resolvidas ou formalmente aceitas como limitações antes de serem tratadas como comportamento definitivo.

## 12. Estratégia de testes e validação

A validação deve demonstrar que a feature resolve a necessidade dos clientes e respeita as decisões e limitações documentadas.

### 12.1. Validação dos cenários de uso

- Cadastrar endpoints para clientes distintos e selecionar diferentes status de interesse.
- Confirmar que somente os endpoints interessados recebem notificações de uma mudança de status.
- Alterar, desativar e remover configurações e verificar o efeito sobre eventos futuros.
- Confirmar que o cliente consegue validar a assinatura da notificação.
- Consultar o histórico de entregas e identificar tentativas bem-sucedidas e malsucedidas.
- Simular indisponibilidade do endpoint, observar as tentativas automáticas e validar o encaminhamento para a DLQ e o replay administrativo.

### 12.2. Validação funcional e de confiabilidade

- Verificar que a mudança de status e o registro do evento são confirmados de forma atômica.
- Verificar que uma falha ao registrar o evento não deixa a mudança de status confirmada sem a notificação correspondente.
- Confirmar que o conteúdo enviado representa o pedido no momento da mudança.
- Verificar que o processamento de notificações não depende do ciclo de vida do processo da API.
- Validar a política de retry, a preservação das falhas permanentes e o controle de acesso ao replay.
- Simular duplicidade e confirmar que o identificador disponibilizado permite a deduplicação pelo consumidor.

### 12.3. Validação não funcional e de segurança

- Medir o tempo de entrega em homologação, utilizando 100 eventos consecutivos e endpoint saudável, verificando p95 inferior a 10 segundos.
- Verificar o comportamento diante de endpoints lentos ou indisponíveis e observar possíveis atrasos para outros consumidores.
- Confirmar o uso obrigatório de HTTPS e a rejeição de payloads acima de 64 KB.
- Validar a assinatura HMAC-SHA256, a exclusividade da credencial por endpoint e o período de convivência de 24 horas na rotação.
- Verificar que credenciais não aparecem em respostas de consulta nem em logs.
- Realizar a revisão de segurança exigida antes do deploy.
- Verificar a disponibilidade de informações de acompanhamento: tentativas, resultados, tempos de resposta, falhas e correlação operacional.

### 12.4. Integração com consumidores externos

Antes da disponibilização, pelo menos um consumidor de homologação deve demonstrar o recebimento e a validação de uma notificação, incluindo:

- validação da assinatura;
- identificação do evento;
- tratamento de notificações duplicadas;
- tratamento de falhas temporárias e recebimento de novas tentativas;
- atualização correta do sistema consumidor a partir da mudança de status.

A documentação para integradores deve deixar explícito que a entrega é *at-least-once*, que duplicatas são possíveis e que a deduplicação é responsabilidade do consumidor.

### 12.5. Prontidão para disponibilização

A feature poderá ser considerada pronta para disponibilização quando:

1. Os critérios de aceitação funcionais e não funcionais aplicáveis tiverem sido verificados.
2. O fluxo de entrega, retry, DLQ e replay tiver sido validado.
3. A integração de homologação tiver demonstrado o recebimento e a validação das notificações.
4. A revisão de segurança prevista na ADR-004 tiver sido concluída.
5. A documentação de integração estiver disponível aos clientes.
6. As pendências que afetam segurança, autorização, semântica de entrega e operação tiverem sido resolvidas ou formalmente aceitas pelos responsáveis.

---

## Referências documentais

Este PRD foi consolidado exclusivamente a partir dos documentos abaixo:

- [ADR-001 — Transactional Outbox com MySQL](./adrs/ADR-001-transactional-outbox-com-mysql.md)
- [ADR-002 — Worker separado, polling e processamento sequencial](./adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md)
- [ADR-003 — Política de retry e Dead Letter Queue](./adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md)
- [ADR-004 — Autenticação e assinatura HMAC-SHA256](./adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md)
- [ADR-005 — Garantia at-least-once e deduplicação por X-Event-Id](./adrs/ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md)
- [ADR-006 — Reutilização dos padrões arquiteturais existentes](./adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md)
- [RFC — Webhooks de saída para mudança de status de pedidos](./RFC.md)
- [FDD — Webhooks de saída para mudança de status de pedidos](./FDD.md)