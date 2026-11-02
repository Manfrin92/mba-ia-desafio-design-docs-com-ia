# Tracker

Este documento relaciona os requisitos, decisões, restrições, alternativas, contratos e critérios de aceite identificados no pacote de design do Sistema de Webhooks de Notificação de Pedidos às evidências primárias disponíveis na transcrição e no código-fonte. Cada ocorrência documental é registrada separadamente, mesmo quando o assunto também aparece em outro documento.

> **Limites da evidência:** o PRD foi incorporado ao Tracker por seus itens identificados e por exclusões/decisões próprias da seção de escopo. `PRD-AC-06` não foi incluído na tabela porque o critério p95 para 100 eventos não tem evidência direta na transcrição nem implementação verificável no código. Em `PRD-FR-02`, `PRD-FR-03`, `PRD-FR-04` e `PRD-NFR-09`, a localização comprova apenas o comportamento central resumido; detalhes adicionais sobre não exposição de secrets, campos específicos de edição, desativação e observabilidade ampliada não foram comprovados integralmente. As seis ADRs estão em `docs/adrs/`. Itens do FDD marcados como inferência ou pendência sem evidência primária não foram apresentados como decisões comprovadas. As referências `CODIGO` demonstram apenas comportamento e padrões existentes, não a implementação dos webhooks.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-FR-01 | `docs/PRD.md` | Requisito Funcional | Cadastrar um endpoint de webhook associado a um cliente, informando a URL de destino e os status de pedido de interesse. | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-02 | `docs/PRD.md` | Requisito Funcional | Consultar os endpoints cadastrados para um cliente; a evidência cobre a consulta, mas não confirma a regra de não exposição de secrets em respostas. | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | `docs/PRD.md` | Requisito Funcional | Permitir atualizar a configuração de um endpoint de webhook. | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | `docs/PRD.md` | Requisito Funcional | Permitir remover a configuração de um endpoint de webhook. | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | `docs/PRD.md` | Regra de Negócio | Permitir selecionar os status de pedido que cada endpoint deseja receber e não gerar eventos para status não selecionados. | TRANSCRICAO | [09:34] Bruno |
| PRD-FR-06 | `docs/PRD.md` | Requisito Funcional | Gerar notificação quando um pedido mudar para um status selecionado por pelo menos um endpoint ativo do cliente. | TRANSCRICAO | [09:34] Bruno |
| PRD-FR-07 | `docs/PRD.md` | Requisito Funcional | Preservar na notificação o pedido e os status anterior e novo correspondentes ao instante da mudança. | TRANSCRICAO | [09:52] Larissa |
| PRD-FR-08 | `docs/PRD.md` | Requisito Funcional | Enviar notificações de forma assíncrona, sem depender da disponibilidade do endpoint externo para confirmar a mudança do pedido. | TRANSCRICAO | [09:04] Bruno |
| PRD-FR-09 | `docs/PRD.md` | Requisito Funcional | Repetir entregas malsucedidas de acordo com a política de backoff e o limite de tentativas. | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-10 | `docs/PRD.md` | Requisito Funcional | Encaminhar eventos após o esgotamento das tentativas para uma área de falhas permanentes (DLQ). | TRANSCRICAO | [09:18] Diego |
| PRD-FR-11 | `docs/PRD.md` | Requisito Funcional | Permitir replay manual de evento na DLQ por ADMIN, registrando quem executou a ação. | TRANSCRICAO | [09:36] Sofia |
| PRD-FR-12 | `docs/PRD.md` | Requisito Funcional | Consultar as últimas 100 tentativas de entrega, incluindo resultado, resposta recebida e tempo de resposta. | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-13 | `docs/PRD.md` | Requisito Funcional | Assinar notificações com HMAC-SHA256 e usar uma credencial exclusiva para cada endpoint. | TRANSCRICAO | [09:22] Sofia |
| PRD-FR-14 | `docs/PRD.md` | Requisito Funcional | Permitir rotação da credencial de um endpoint, mantendo a credencial anterior válida por 24 horas. | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-15 | `docs/PRD.md` | Requisito Funcional | Disponibilizar identificador único do evento ao consumidor para reconhecimento e deduplicação de notificações repetidas. | TRANSCRICAO | [09:25] Diego |
| PRD-NFR-01 | `docs/PRD.md` | Requisito Não Funcional | Atender à necessidade de negócio de entrega das notificações em menos de 10 segundos; o critério p95/100 eventos do PRD não tem evidência primária específica. | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-02 | `docs/PRD.md` | Requisito Não Funcional | Manter atomicidade entre a mudança de status do pedido e o registro do evento correspondente. | TRANSCRICAO | [09:40] Bruno |
| PRD-NFR-03 | `docs/PRD.md` | Requisito Não Funcional | Adotar entrega at-least-once e exigir deduplicação do lado do consumidor pelo identificador do evento. | TRANSCRICAO | [09:25] Diego |
| PRD-NFR-04 | `docs/PRD.md` | Requisito Não Funcional | Limitar retentativas e encaminhar eventos que esgotem a política para DLQ, com recuperação administrativa. | TRANSCRICAO | [09:18] Diego |
| PRD-NFR-05 | `docs/PRD.md` | Requisito Não Funcional | Usar HTTPS nos destinos e assinatura HMAC-SHA256 com credencial exclusiva por endpoint. | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-06 | `docs/PRD.md` | Requisito Não Funcional | Rejeitar payloads de webhook que ultrapassem o limite de 64 KB, sem truncamento. | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-07 | `docs/PRD.md` | Requisito Não Funcional | Permitir rotação de credencial com convivência de 24 horas entre a antiga e a nova. | TRANSCRICAO | [09:21] Sofia |
| PRD-NFR-08 | `docs/PRD.md` | Requisito Não Funcional | Reservar pelo menos dois dias úteis para revisão de segurança do código de HMAC e geração de credenciais antes do deploy. | TRANSCRICAO | [09:46] Sofia |
| PRD-NFR-09 | `docs/PRD.md` | Requisito Não Funcional | Acompanhar entregas pelo histórico recente, incluindo resultado e tempo de resposta; métricas, correlação e logging específico da feature não foram comprovados. | TRANSCRICAO | [09:34] Marcos |
| PRD-NFR-10 | `docs/PRD.md` | Requisito Não Funcional | Executar o processamento de notificações em worker separado do processo da API. | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-11 | `docs/PRD.md` | Requisito Não Funcional | Manter a primeira versão com um único worker e processamento sequencial, sem paralelismo nesta fase. | TRANSCRICAO | [09:13] Diego |
| PRD-AC-01 | `docs/PRD.md` | Critério de Aceitação | Cadastrar endpoint de cliente com URL e status de interesse por usuário autenticado, gerando a credencial de integração. | TRANSCRICAO | [09:31] Marcos |
| PRD-AC-02 | `docs/PRD.md` | Critério de Aceitação | Consultar, atualizar e remover configurações de webhook; as alterações de configuração devem valer para eventos futuros. | TRANSCRICAO | [09:33] Bruno |
| PRD-AC-03 | `docs/PRD.md` | Critério de Aceitação | Enviar notificação quando o pedido mudar para status selecionado por endpoint ativo. | TRANSCRICAO | [09:34] Bruno |
| PRD-AC-04 | `docs/PRD.md` | Critério de Aceitação | Manter o conteúdo da notificação correspondente aos dados do pedido no momento da mudança de status. | TRANSCRICAO | [09:52] Larissa |
| PRD-AC-05 | `docs/PRD.md` | Critério de Aceitação | Impedir que lentidão ou indisponibilidade do endpoint externo bloqueie a confirmação da mudança de status. | TRANSCRICAO | [09:04] Bruno |
| PRD-AC-07 | `docs/PRD.md` | Critério de Aceitação | Executar retries segundo a política definida e disponibilizar na DLQ os eventos que esgotarem as tentativas. | TRANSCRICAO | [09:18] Diego |
| PRD-AC-08 | `docs/PRD.md` | Critério de Aceitação | Restringir o replay a ADMIN e registrar a identidade de quem executou a operação. | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-09 | `docs/PRD.md` | Critério de Aceitação | Consultar as últimas 100 tentativas por endpoint com identificação do evento, resultado, resposta e duração. | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-10 | `docs/PRD.md` | Critério de Aceitação | Assinar cada notificação com HMAC-SHA256 e credencial exclusiva do endpoint correspondente. | TRANSCRICAO | [09:22] Sofia |
| PRD-AC-11 | `docs/PRD.md` | Critério de Aceitação | Manter a credencial anterior válida por 24 horas após rotação e disponibilizar a nova credencial ao solicitante. | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-12 | `docs/PRD.md` | Critério de Aceitação | Fornecer identificador estável para deduplicação pelo consumidor e documentar que entregas duplicadas podem ocorrer. | TRANSCRICAO | [09:25] Diego |
| PRD-AC-13 | `docs/PRD.md` | Critério de Aceitação | Concluir a revisão de segurança de HMAC e geração de credenciais com pelo menos dois dias úteis antes do deploy. | TRANSCRICAO | [09:46] Sofia |
| PRD-SCOPE-OUT-01 | `docs/PRD.md` | Fora de Escopo | A feature contempla webhooks de saída do OMS; webhooks de entrada enviados por clientes não fazem parte desta entrega. | TRANSCRICAO | [09:02] Marcos |
| PRD-SCOPE-OUT-02 | `docs/PRD.md` | Fora de Escopo | O arquivamento de eventos entregues após aproximadamente 30 dias foi excluído desta entrega. | TRANSCRICAO | [09:08] Diego |
| PRD-SCOPE-OUT-03 | `docs/PRD.md` | Trade-off | A garantia exactly-once e a deduplicação no servidor foram rejeitadas em favor de at-least-once com deduplicação pelo consumidor. | TRANSCRICAO | [09:25] Diego |
| PRD-SCOPE-OUT-04 | `docs/PRD.md` | Alternativa | A chamada HTTP síncrona durante a transação de mudança de status foi rejeitada. | TRANSCRICAO | [09:04] Bruno |
| PRD-SCOPE-OUT-05 | `docs/PRD.md` | Fora de Escopo | Avisos por e-mail ao cliente em caso de falha foram adiados para fase futura. | TRANSCRICAO | [09:37] Larissa |
| PRD-SCOPE-OUT-06 | `docs/PRD.md` | Restrição | Rate limiting de envio por cliente foi adiado; o comportamento será observado antes de definir limites. | TRANSCRICAO | [09:39] Larissa |
| PRD-SCOPE-OUT-07 | `docs/PRD.md` | Restrição | A primeira versão não executará múltiplos workers em paralelo. | TRANSCRICAO | [09:13] Diego |
| PRD-SCOPE-OUT-08 | `docs/PRD.md` | Questão em Aberto | O endurecimento da autorização para restringir gestão ao próprio cliente permaneceu pendente; foi aceita provisoriamente qualquer role autenticada. | TRANSCRICAO | [09:37] Sofia |
| PRD-DEC-01 | `docs/PRD.md` | Decisão | Reutilizar a estrutura modular, os padrões, as bibliotecas e a infraestrutura já existentes no OMS. | TRANSCRICAO | [09:30] Larissa |
| RFC-CTX-01 | `docs/RFC.md` | Contexto | Clientes B2B consultam GET /orders periodicamente para descobrir mudanças de status. | TRANSCRICAO | [09:00] Marcos |
| RFC-REQ-01 | `docs/RFC.md` | Requisito Não Funcional | A notificação deve chegar em menos de 10 segundos para ser considerada tempo real. | TRANSCRICAO | [09:02] Marcos |
| RFC-SCOPE-01 | `docs/RFC.md` | Restrição | A feature cobre somente webhooks de saída do OMS para os clientes. | TRANSCRICAO | [09:02] Marcos |
| RFC-DEC-01 | `docs/RFC.md` | Decisão | Registrar eventos em uma outbox no MySQL existente, na mesma transação da mudança de status. | TRANSCRICAO | [09:08] Larissa |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa | A chamada HTTP síncrona dentro da transação foi rejeitada por acoplar a mudança de status à disponibilidade do cliente. | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | `docs/RFC.md` | Trade-off | Redis Streams foi descartado para evitar infraestrutura adicional considerada excessiva para o tamanho do time. | TRANSCRICAO | [09:07] Diego |
| RFC-DEC-02 | `docs/RFC.md` | Decisão | O worker consulta a outbox por polling a cada 2 segundos. | TRANSCRICAO | [09:10] Larissa |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa | Trigger não resolve a notificação de um processo externo e LISTEN/NOTIFY não está disponível no MySQL. | TRANSCRICAO | [09:09] Diego |
| RFC-DEC-03 | `docs/RFC.md` | Decisão | O worker deve executar em processo separado do processo da API. | TRANSCRICAO | [09:11] Diego |
| RFC-DEC-04 | `docs/RFC.md` | Decisão | O processamento inicial será sequencial, com um único worker e ordem por criação dos eventos. | TRANSCRICAO | [09:12] Diego |
| RFC-RESTR-01 | `docs/RFC.md` | Restrição | A ordenação não é global; a limitação por pedido depende de manter um único worker. | TRANSCRICAO | [09:13] Larissa |
| RFC-OPEN-01 | `docs/RFC.md` | Questão em Aberto | Escalar para múltiplos workers, particionando por order_id ou usando lock, ficou para uma fase futura. | TRANSCRICAO | [09:13] Diego |
| RFC-DEC-05 | `docs/RFC.md` | Decisão | As falhas terão retry com backoff exponencial e limite de cinco tentativas. | TRANSCRICAO | [09:17] Larissa |
| RFC-DEC-06 | `docs/RFC.md` | Decisão | A progressão definida para o backoff é 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. | TRANSCRICAO | [09:17] Diego |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa | A proposta de três tentativas foi rejeitada por não cobrir indisponibilidades mais longas. | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-05 | `docs/RFC.md` | Alternativa | Retry indefinido foi rejeitado para evitar eventos pendurados sem limite quando o cliente desaparece. | TRANSCRICAO | [09:15] Diego |
| RFC-DEC-07 | `docs/RFC.md` | Decisão | Após o esgotamento das tentativas, o evento vai para uma DLQ em tabela separada. | TRANSCRICAO | [09:18] Diego |
| RFC-DEC-08 | `docs/RFC.md` | Decisão | O replay de itens da DLQ será manual, por endpoint administrativo que recoloca o evento como pendente. | TRANSCRICAO | [09:18] Diego |
| RFC-DEC-09 | `docs/RFC.md` | Decisão | O envio será assinado com HMAC-SHA256 calculado sobre o corpo da requisição. | TRANSCRICAO | [09:22] Sofia |
| RFC-DEC-10 | `docs/RFC.md` | Decisão | Cada endpoint de webhook terá uma secret própria, em vez de uma secret global da plataforma. | TRANSCRICAO | [09:21] Sofia |
| RFC-DEC-11 | `docs/RFC.md` | Decisão | A rotação da secret mantém a secret anterior válida em paralelo por 24 horas. | TRANSCRICAO | [09:21] Sofia |
| RFC-ALT-06 | `docs/RFC.md` | Alternativa | Exactly-once foi rejeitado por exigir coordenação entre plataforma e cliente e aumentar a complexidade. | TRANSCRICAO | [09:25] Diego |
| RFC-DEC-12 | `docs/RFC.md` | Decisão | A entrega será at-least-once e o cliente deduplicará usando o identificador enviado em X-Event-Id. | TRANSCRICAO | [09:26] Larissa |
| RFC-DEC-13 | `docs/RFC.md` | Decisão | A implementação deve reutilizar os padrões existentes de módulos, erros, logging, validação e autenticação. | TRANSCRICAO | [09:30] Larissa |
| RFC-DEC-14 | `docs/RFC.md` | Decisão | A API de gestão usa JWT e aceita qualquer role autenticada nesta fase; o replay exige ADMIN. | TRANSCRICAO | [09:36] Larissa |
| RFC-REQ-02 | `docs/RFC.md` | Requisito Funcional | O cliente pode cadastrar um webhook com URL, secret gerada pela plataforma e lista de status de interesse. | TRANSCRICAO | [09:31] Marcos |
| RFC-REQ-03 | `docs/RFC.md` | Requisito Funcional | A API deve permitir editar, remover e listar webhooks de um customer. | TRANSCRICAO | [09:33] Bruno |
| RFC-REQ-04 | `docs/RFC.md` | Regra de Negócio | O endpoint configura os status de pedido que deseja receber. | TRANSCRICAO | [09:33] Marcos |
| RFC-DEC-15 | `docs/RFC.md` | Decisão | O filtro por status é aplicado na inserção da outbox para não criar eventos sem endpoint interessado. | TRANSCRICAO | [09:34] Bruno |
| RFC-REQ-05 | `docs/RFC.md` | Requisito Funcional | O cliente pode consultar os últimos 100 registros de entrega, incluindo sucesso ou falha, payload, resposta e tempo de resposta. | TRANSCRICAO | [09:34] Marcos |
| RFC-REQ-06 | `docs/RFC.md` | Requisito Funcional | O replay administrativo deve registrar a identidade de quem o executou para auditoria. | TRANSCRICAO | [09:36] Sofia |
| RFC-OPEN-02 | `docs/RFC.md` | Restrição | Avisos por e-mail em caso de falha foram adiados para uma fase futura. | TRANSCRICAO | [09:37] Larissa |
| RFC-OPEN-03 | `docs/RFC.md` | Questão em Aberto | Rate limiting de envio por cliente será observado e decidido posteriormente. | TRANSCRICAO | [09:39] Larissa |
| RFC-OUT-01 | `docs/RFC.md` | Restrição | Dashboard visual para clientes ficou fora do escopo; a interação será por endpoints. | TRANSCRICAO | [09:40] Larissa |
| RFC-DEC-16 | `docs/RFC.md` | Decisão | A publicação do evento será feita por publishWebhookEvent, recebendo o client da transação de changeStatus. | TRANSCRICAO | [09:41] Bruno |
| RFC-REQ-07 | `docs/RFC.md` | Requisito Não Funcional | A chamada HTTP do worker terá timeout de 10 segundos; timeout será tratado como falha e agendado para retry. | TRANSCRICAO | [09:42] Diego |
| RFC-REQ-08 | `docs/RFC.md` | Contrato | O payload JSON inclui identificadores, tipo e timestamp do evento, pedido, customer e status anterior e novo; não inclui items. | TRANSCRICAO | [09:43] Diego |
| RFC-REQ-09 | `docs/RFC.md` | Contrato | A entrega inclui X-Event-Id, X-Signature, X-Timestamp e Content-Type application/json. | TRANSCRICAO | [09:44] Diego |
| RFC-REQ-10 | `docs/RFC.md` | Contrato | A entrega também inclui X-Webhook-Id para identificar qual cadastro de webhook recebeu o evento. | TRANSCRICAO | [09:44] Sofia |
| RFC-REQ-11 | `docs/RFC.md` | Restrição | A URL cadastrada deve usar HTTPS; URLs HTTP serão recusadas. | TRANSCRICAO | [09:23] Sofia |
| RFC-REQ-12 | `docs/RFC.md` | Requisito Não Funcional | Payloads acima de 64 KB devem gerar erro, sem truncamento. | TRANSCRICAO | [09:24] Larissa |
| RFC-RESTR-02 | `docs/RFC.md` | Restrição | O customer_id da gestão é informado no body ou path, não inferido do JWT atual. | TRANSCRICAO | [09:32] Larissa |
| RFC-OUT-02 | `docs/RFC.md` | Restrição | O arquivamento de eventos entregues após aproximadamente 30 dias ficou fora desta feature. | TRANSCRICAO | [09:08] Diego |
| RFC-PLAN-01 | `docs/RFC.md` | Restrição | A estimativa apresentada para a feature foi de três sprints, incluindo revisão de segurança. | TRANSCRICAO | [09:46] Larissa |
| RFC-SEC-01 | `docs/RFC.md` | Restrição | A revisão de segurança de HMAC e geração de secret deve reservar pelo menos dois dias úteis antes do deploy. | TRANSCRICAO | [09:46] Sofia |
| RFC-DEC-17 | `docs/RFC.md` | Decisão | O identificador da outbox seguirá o padrão UUID existente no projeto. | TRANSCRICAO | [09:51] Larissa |
| RFC-DEC-18 | `docs/RFC.md` | Decisão | O payload será materializado como snapshot no momento da inserção na outbox. | TRANSCRICAO | [09:52] Larissa |
| ADR-001-DEC-01 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Decisão | Usar uma outbox no MySQL existente para registrar o evento na transação de mudança do pedido. | TRANSCRICAO | [09:06] Diego |
| ADR-001-DEC-02 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Decisão | A inserção da outbox e a mudança de status devem ser atômicas; falha na outbox provoca rollback. | TRANSCRICAO | [09:40] Bruno |
| ADR-001-DEC-03 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Decisão | O OrderService chamará publishWebhookEvent(tx, order, fromStatus, toStatus), recebendo o client transacional. | TRANSCRICAO | [09:41] Bruno |
| ADR-001-DEC-04 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Decisão | A outbox terá estados para pendente, processando, falhou e entregue, com índices de status e created_at. | TRANSCRICAO | [09:08] Diego |
| ADR-001-DEC-05 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Decisão | O filtro dos status inscritos será aplicado ao inserir eventos na outbox. | TRANSCRICAO | [09:34] Bruno |
| ADR-001-DEC-06 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Decisão | O payload completo do evento será salvo como snapshot na inserção, para refletir o estado do pedido naquele momento. | TRANSCRICAO | [09:52] Larissa |
| ADR-001-DEC-07 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Decisão | O identificador da outbox usará UUID, seguindo o padrão do projeto. | TRANSCRICAO | [09:51] Larissa |
| ADR-001-RESTR-01 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Restrição | O arquivamento das linhas entregues após cerca de 30 dias não faz parte desta feature. | TRANSCRICAO | [09:08] Diego |
| ADR-001-ALT-01 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Alternativa | A chamada HTTP síncrona dentro do OrderService foi rejeitada por travar a transação quando o cliente demora ou está indisponível. | TRANSCRICAO | [09:04] Bruno |
| ADR-001-ALT-02 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Alternativa | Redis Streams foi rejeitado por exigir infraestrutura adicional para um time pequeno. | TRANSCRICAO | [09:07] Diego |
| ADR-001-CODE-01 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Integração com Código | OrderService.changeStatus já executa update do pedido e criação do histórico dentro de uma transação Prisma. | CODIGO | `src/modules/orders/order.service.ts` |
| ADR-001-CODE-02 | `docs/adrs/ADR-001-transactional-outbox-com-mysql.md` | Integração com Código | A máquina de estados do pedido já define transições válidas e estados terminais. | CODIGO | `src/modules/orders/order.status.ts` |
| ADR-002-DEC-01 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Decisão | O worker será executado em um processo separado da API, com entry point próprio. | TRANSCRICAO | [09:11] Diego |
| ADR-002-DEC-02 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Decisão | O worker usará o mesmo banco e DATABASE_URL, mas terá sua própria instância PrismaClient por processo. | TRANSCRICAO | [09:30] Bruno |
| ADR-002-DEC-03 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Decisão | O polling será executado a cada 2 segundos. | TRANSCRICAO | [09:09] Diego |
| ADR-002-DEC-04 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Decisão | O worker buscará os eventos pendentes mais antigos, processando-os em sequência. | TRANSCRICAO | [09:09] Diego |
| ADR-002-DEC-05 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Decisão | A primeira versão usará um único worker e processamento sequencial por ordem de criação. | TRANSCRICAO | [09:12] Diego |
| ADR-002-RESTR-01 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Restrição | A ordenação por pedido não é garantia global e deixa de ser garantida ao escalar para múltiplos workers. | TRANSCRICAO | [09:13] Larissa |
| ADR-002-DEC-06 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Decisão | A chamada HTTP terá timeout de 10 segundos e timeout será tratado como falha para retry. | TRANSCRICAO | [09:42] Diego |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Alternativa | Trigger ou mecanismo similar no banco foi rejeitado por não notificar diretamente um processo externo. | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-02 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Alternativa | Manter o worker dentro do processo da API foi rejeitado porque um restart da API também derrubaria o worker. | TRANSCRICAO | [09:11] Diego |
| ADR-002-OPEN-01 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Questão em Aberto | Particionamento por order_id ou lock pessimista para futura escala com múltiplos workers foi adiado. | TRANSCRICAO | [09:13] Diego |
| ADR-002-CODE-01 | `docs/adrs/ADR-002-worker-separado-polling-e-processamento-sequencial.md` | Integração com Código | A aplicação já possui uma fábrica createPrismaClient que instancia PrismaClient para cada processo. | CODIGO | `src/config/database.ts` |
| ADR-003-DEC-01 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Decisão | Aplicar retry exponencial com limite de cinco tentativas. | TRANSCRICAO | [09:17] Larissa |
| ADR-003-DEC-02 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Decisão | A progressão de retry é 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. | TRANSCRICAO | [09:17] Diego |
| ADR-003-DEC-03 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Decisão | Após o limite de tentativas, a falha é permanente e o evento será movido para a DLQ. | TRANSCRICAO | [09:15] Diego |
| ADR-003-DEC-04 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Decisão | A DLQ será armazenada em tabela separada com payload, motivo da falha e timestamp. | TRANSCRICAO | [09:18] Diego |
| ADR-003-DEC-05 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Decisão | O replay será manual por endpoint administrativo e recolocará o evento na outbox como pendente. | TRANSCRICAO | [09:18] Diego |
| ADR-003-DEC-06 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Decisão | O replay exige role ADMIN. | TRANSCRICAO | [09:36] Sofia |
| ADR-003-DEC-07 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Requisito de Auditoria | O endpoint de replay deve registrar quem executou a ação para auditoria. | TRANSCRICAO | [09:36] Sofia |
| ADR-003-OUT-01 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Restrição | Avisos por e-mail ao cliente em caso de falha foram adiados para uma fase futura. | TRANSCRICAO | [09:37] Larissa |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Alternativa | Três tentativas foram rejeitadas por não cobrir indisponibilidade de clientes de maior duração. | TRANSCRICAO | [09:16] Diego |
| ADR-003-ALT-02 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Alternativa | Retry indefinido foi rejeitado para evitar eventos pendentes sem limite. | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-03 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Trade-off | Uma tabela de DLQ separada foi escolhida em vez de marcar a falha permanente na própria outbox. | TRANSCRICAO | [09:18] Diego |
| ADR-003-CODE-01 | `docs/adrs/ADR-003-politica-de-retry-e-dead-letter-queue.md` | Integração com Código | O middleware de autorização já fornece requireRole para restringir rotas a uma role como ADMIN. | CODIGO | `src/middlewares/auth.middleware.ts` |
| ADR-004-DEC-01 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Decisão | Assinar o corpo da requisição de webhook com HMAC-SHA256 e enviar a assinatura em X-Signature. | TRANSCRICAO | [09:22] Sofia |
| ADR-004-DEC-02 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Decisão | Cada endpoint de webhook terá uma secret exclusiva, sem secret global compartilhada por todos os clientes. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-DEC-03 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Decisão | A plataforma gera a secret no cadastro e a devolve ao cliente. | TRANSCRICAO | [09:31] Marcos |
| ADR-004-DEC-04 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Decisão | A rotação mantém a secret anterior válida por 24 horas para permitir a migração do cliente. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-DEC-05 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Contrato | Os envios incluem X-Event-Id, X-Signature, X-Timestamp e Content-Type application/json. | TRANSCRICAO | [09:44] Diego |
| ADR-004-DEC-06 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Contrato | Os envios incluem X-Webhook-Id para identificar o endpoint associado à entrega. | TRANSCRICAO | [09:44] Sofia |
| ADR-004-DEC-07 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Decisão | A API de gestão usa o JWT do sistema e aceita qualquer role autenticada nesta fase. | TRANSCRICAO | [09:37] Sofia |
| ADR-004-DEC-08 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Restrição | O customer_id é informado no body ou path; não é obtido do JWT atual. | TRANSCRICAO | [09:32] Larissa |
| ADR-004-REQ-01 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Requisito Não Funcional | A URL do webhook deve usar HTTPS; URL HTTP será recusada na validação. | TRANSCRICAO | [09:23] Sofia |
| ADR-004-REQ-02 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Requisito Não Funcional | O payload tem limite de 64 KB e deve gerar erro caso ultrapasse esse limite. | TRANSCRICAO | [09:24] Larissa |
| ADR-004-RESTR-01 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Restrição | A revisão do código de HMAC e geração de secrets requer pelo menos dois dias úteis antes do deploy. | TRANSCRICAO | [09:46] Sofia |
| ADR-004-ALT-01 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Alternativa | Uma secret global foi rejeitada porque seu vazamento comprometeria todos os endpoints. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CODE-01 | `docs/adrs/ADR-004-autenticacao-e-assinatura-hmac-sha256.md` | Integração com Código | O middleware existente autentica JWT e expõe as roles ADMIN e OPERATOR para autorização. | CODIGO | `src/middlewares/auth.middleware.ts` |
| ADR-005-DEC-01 | `docs/adrs/ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md` | Decisão | A semântica de entrega é at-least-once; o cliente pode receber o mesmo evento mais de uma vez. | TRANSCRICAO | [09:24] Diego |
| ADR-005-DEC-02 | `docs/adrs/ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md` | Decisão | Cada evento recebe um UUID único, enviado ao cliente no header X-Event-Id. | TRANSCRICAO | [09:25] Diego |
| ADR-005-DEC-03 | `docs/adrs/ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md` | Responsabilidade | A deduplicação pelo event_id será feita no sistema do cliente, não pela plataforma. | TRANSCRICAO | [09:25] Diego |
| ADR-005-DEC-04 | `docs/adrs/ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md` | Requisito de Documentação | A semântica at-least-once e a deduplicação serão destacadas na documentação do portal de desenvolvedores. | TRANSCRICAO | [09:26] Marcos |
| ADR-005-ALT-01 | `docs/adrs/ADR-005-garantia-at-least-once-e-deduplicacao-por-x-event-id.md` | Alternativa | Exactly-once foi rejeitado por exigir coordenação entre os dois lados e maior complexidade. | TRANSCRICAO | [09:25] Diego |
| ADR-006-DEC-01 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Decisão | O módulo de webhooks seguirá a organização por domínio já usada nos módulos existentes. | TRANSCRICAO | [09:27] Bruno |
| ADR-006-DEC-02 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Decisão | O worker terá entry point próprio e a lógica ficará em um módulo de webhooks. | TRANSCRICAO | [09:28] Bruno |
| ADR-006-DEC-03 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Decisão | Os erros do módulo seguirão o padrão AppError e usarão códigos com prefixo WEBHOOK_. | TRANSCRICAO | [09:29] Larissa |
| ADR-006-DEC-04 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Decisão | O módulo reutilizará o logger Pino existente, sem introduzir outro logger. | TRANSCRICAO | [09:29] Bruno |
| ADR-006-DEC-05 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Decisão | As rotas do módulo reutilizarão o middleware centralizado de tratamento de erros existente. | TRANSCRICAO | [09:29] Bruno |
| ADR-006-DEC-06 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Decisão | A validação dos contratos usará schemas Zod seguindo o padrão existente. | TRANSCRICAO | [09:30] Larissa |
| ADR-006-DEC-07 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Decisão | A autorização reaproveitará authenticate e requireRole, exigindo ADMIN no replay. | TRANSCRICAO | [09:36] Larissa |
| ADR-006-DEC-08 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Decisão | O worker usará o mesmo banco e a mesma stack Prisma, com uma instância de cliente por processo. | TRANSCRICAO | [09:30] Bruno |
| ADR-006-ALT-01 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Trade-off | Foi decidido não introduzir infraestrutura ou bibliotecas próprias para fila, logging ou tratamento de erro. | TRANSCRICAO | [09:29] Bruno |
| ADR-006-CODE-01 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Integração com Código | A classe AppError existente padroniza status HTTP, código de erro e detalhes. | CODIGO | `src/shared/errors/app-error.ts` |
| ADR-006-CODE-02 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Integração com Código | As subclasses HTTP existentes reutilizam AppError; algumas permitem código customizado e outras fixam o código. | CODIGO | `src/shared/errors/http-errors.ts` |
| ADR-006-CODE-03 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Integração com Código | O errorMiddleware central trata AppError, ZodError e erros Prisma conhecidos. | CODIGO | `src/middlewares/error.middleware.ts` |
| ADR-006-CODE-04 | `docs/adrs/ADR-006-reutilizacao-dos-padroes-arquiteturais-existentes.md` | Integração com Código | O logger compartilhado é implementado com Pino e já possui regras de redação de campos sensíveis. | CODIGO | `src/shared/logger/index.ts` |
| FDD-OT-01 | `docs/FDD.md` | Objetivo Técnico | Gravar o evento na mesma transação de changeStatus; falha ao inserir na outbox deve reverter a mudança de status. | TRANSCRICAO | [09:40] Bruno |
| FDD-OT-02 | `docs/FDD.md` | Objetivo Técnico | Salvar o payload já renderizado como snapshot no momento da inserção. | TRANSCRICAO | [09:52] Larissa |
| FDD-OT-03 | `docs/FDD.md` | Objetivo Técnico | Aplicar o filtro de status do webhook na inserção da outbox. | TRANSCRICAO | [09:34] Bruno |
| FDD-OT-04 | `docs/FDD.md` | Objetivo Técnico | Executar o worker em processo separado e consultar a outbox por polling de 2 segundos. | TRANSCRICAO | [09:11] Diego |
| FDD-OT-05 | `docs/FDD.md` | Objetivo Técnico | Processar eventos em sequência, ordenados pela criação na outbox. | TRANSCRICAO | [09:12] Diego |
| FDD-OT-06 | `docs/FDD.md` | Objetivo Técnico | Aplicar retry com backoff de 1m, 5m, 30m, 2h e 12h, limitado a cinco tentativas conforme a decisão registrada. | TRANSCRICAO | [09:17] Larissa |
| FDD-OT-07 | `docs/FDD.md` | Objetivo Técnico | Mover eventos com falha permanente para uma DLQ separada e permitir replay manual. | TRANSCRICAO | [09:18] Diego |
| FDD-OT-08 | `docs/FDD.md` | Objetivo Técnico | Restringir replay a ADMIN e registrar a identidade de quem executou a ação. | TRANSCRICAO | [09:36] Sofia |
| FDD-OT-09 | `docs/FDD.md` | Objetivo Técnico | Assinar o corpo com HMAC-SHA256, usando secret exclusiva por endpoint com rotação e convivência por 24 horas. | TRANSCRICAO | [09:22] Sofia |
| FDD-OT-10 | `docs/FDD.md` | Objetivo Técnico | Enviar os headers de identificação do evento, assinatura, timestamp e endpoint de webhook. | TRANSCRICAO | [09:44] Diego |
| FDD-OT-11 | `docs/FDD.md` | Objetivo Técnico | Disponibilizar API autenticada para CRUD de webhooks, rotação de secret e consulta de entregas. | TRANSCRICAO | [09:33] Bruno |
| FDD-SCOPE-01 | `docs/FDD.md` | Funcionalidade | Disparar webhook de saída quando o status do pedido muda. | TRANSCRICAO | [09:00] Marcos |
| FDD-SCOPE-02 | `docs/FDD.md` | Funcionalidade | Cadastrar endpoints de webhook para customers pela API. | TRANSCRICAO | [09:31] Marcos |
| FDD-SCOPE-03 | `docs/FDD.md` | Funcionalidade | Permitir listar, editar e remover os endpoints de webhook. | TRANSCRICAO | [09:33] Bruno |
| FDD-SCOPE-04 | `docs/FDD.md` | Regra de Negócio | Permitir configurar os status específicos que cada endpoint deseja receber. | TRANSCRICAO | [09:33] Marcos |
| FDD-SCOPE-05 | `docs/FDD.md` | Funcionalidade | Permitir consulta do histórico dos últimos 100 registros de entrega. | TRANSCRICAO | [09:34] Marcos |
| FDD-SCOPE-06 | `docs/FDD.md` | Funcionalidade | Permitir replay manual de eventos registrados na DLQ. | TRANSCRICAO | [09:18] Diego |
| FDD-SCOPE-07 | `docs/FDD.md` | Funcionalidade | Permitir solicitar a rotação da secret de cada endpoint. | TRANSCRICAO | [09:21] Sofia |
| FDD-SCOPE-08 | `docs/FDD.md` | Dependência | Documentar para os clientes a integração por API e a semântica de deduplicação. | TRANSCRICAO | [09:26] Marcos |
| FDD-OUT-01 | `docs/FDD.md` | Fora de Escopo | Webhooks de entrada, enviados pelos clientes para o OMS, não fazem parte da feature. | TRANSCRICAO | [09:02] Marcos |
| FDD-OUT-02 | `docs/FDD.md` | Fora de Escopo | Arquivamento de eventos entregues após aproximadamente 30 dias foi adiado. | TRANSCRICAO | [09:08] Diego |
| FDD-OUT-03 | `docs/FDD.md` | Fora de Escopo | Avisos por e-mail ao cliente quando houver falha foram adiados para outra fase. | TRANSCRICAO | [09:37] Larissa |
| FDD-OUT-04 | `docs/FDD.md` | Fora de Escopo | Rate limiting de envio por customer não será implementado nesta fase; o impacto será observado. | TRANSCRICAO | [09:39] Larissa |
| FDD-OUT-05 | `docs/FDD.md` | Restrição | A primeira versão não terá múltiplos workers em paralelo. | TRANSCRICAO | [09:13] Diego |
| FDD-OUT-06 | `docs/FDD.md` | Fora de Escopo | Dashboard visual para clientes não será entregue; a feature terá endpoints de API. | TRANSCRICAO | [09:40] Larissa |
| FDD-OUT-07 | `docs/FDD.md` | Restrição | A plataforma não implementará exactly-once nem deduplicação do lado do servidor; o cliente deduplica pelo event_id. | TRANSCRICAO | [09:25] Diego |
| FDD-OUT-08 | `docs/FDD.md` | Restrição | O endurecimento da autorização de gestão além de qualquer role autenticada foi deixado para o futuro. | TRANSCRICAO | [09:37] Sofia |
| FDD-E1 | `docs/FDD.md` | Contrato | POST de cadastro de webhook recebe URL, secret gerada pela plataforma e filtros de status. | TRANSCRICAO | [09:31] Marcos |
| FDD-E2 | `docs/FDD.md` | Contrato | GET lista os webhooks cadastrados para um customer. | TRANSCRICAO | [09:33] Bruno |
| FDD-E4 | `docs/FDD.md` | Contrato | PATCH atualiza a configuração do endpoint de webhook. | TRANSCRICAO | [09:33] Bruno |
| FDD-E5 | `docs/FDD.md` | Contrato | DELETE remove a configuração do webhook. | TRANSCRICAO | [09:33] Bruno |
| FDD-E6 | `docs/FDD.md` | Contrato | A API permite solicitar nova secret para um endpoint de webhook. | TRANSCRICAO | [09:21] Sofia |
| FDD-E7 | `docs/FDD.md` | Contrato | GET de deliveries retorna os últimos 100 registros com sucesso/falha, payload, resposta e tempo de resposta. | TRANSCRICAO | [09:34] Marcos |
| FDD-E9 | `docs/FDD.md` | Contrato | POST administrativo executa replay manual de um item da DLQ. | TRANSCRICAO | [09:18] Diego |
| FDD-AC-01 | `docs/FDD.md` | Critério de Aceitação | Mudança de status e inserção dos eventos interessados na outbox devem ocorrer no mesmo commit. | TRANSCRICAO | [09:40] Bruno |
| FDD-AC-02 | `docs/FDD.md` | Critério de Aceitação | Se a inserção na outbox falhar, a mudança do pedido deve sofrer rollback. | TRANSCRICAO | [09:41] Diego |
| FDD-AC-03 | `docs/FDD.md` | Critério de Aceitação | O payload salvo deve refletir o estado do pedido no momento da mudança e não incluir items. | TRANSCRICAO | [09:52] Larissa |
| FDD-AC-04 | `docs/FDD.md` | Critério de Aceitação | Nenhuma linha de outbox deve ser criada se não houver endpoint inscrito para o status. | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-05 | `docs/FDD.md` | Critério de Aceitação | O worker deve funcionar independentemente do processo da API. | TRANSCRICAO | [09:11] Diego |
| FDD-AC-06 | `docs/FDD.md` | Critério de Aceitação | A meta de entrega é inferior a 10 segundos, com polling configurado em 2 segundos. | TRANSCRICAO | [09:02] Marcos |
| FDD-AC-07 | `docs/FDD.md` | Critério de Aceitação | Uma tentativa sem resposta após 10 segundos deve ser tratada como falha para retry. | TRANSCRICAO | [09:42] Diego |
| FDD-AC-08 | `docs/FDD.md` | Critério de Aceitação | As retentativas devem respeitar a progressão de 1m, 5m, 30m, 2h e 12h. | TRANSCRICAO | [09:17] Diego |
| FDD-AC-09 | `docs/FDD.md` | Critério de Aceitação | Após esgotar as tentativas, o evento deve ser transferido para uma tabela DLQ separada. | TRANSCRICAO | [09:18] Diego |
| FDD-AC-11 | `docs/FDD.md` | Critério de Aceitação | Somente ADMIN pode solicitar replay; a ação deve registrar quem a executou. | TRANSCRICAO | [09:36] Sofia |
| FDD-AC-12 | `docs/FDD.md` | Critério de Aceitação | Cada entrega deve conter assinatura HMAC-SHA256 calculada sobre o corpo enviado. | TRANSCRICAO | [09:22] Sofia |
| FDD-AC-13 | `docs/FDD.md` | Critério de Aceitação | A secret anterior permanece válida por 24 horas após a rotação. | TRANSCRICAO | [09:21] Sofia |
| FDD-AC-14 | `docs/FDD.md` | Critério de Aceitação | A entrega inclui X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id e Content-Type application/json. | TRANSCRICAO | [09:44] Sofia |
| FDD-AC-15 | `docs/FDD.md` | Critério de Aceitação | O CRUD de configuração usa JWT; qualquer role autenticada pode gerenciar endpoints nesta fase. | TRANSCRICAO | [09:37] Sofia |
| FDD-AC-16 | `docs/FDD.md` | Critério de Aceitação | O histórico de entregas apresenta até 100 registros, com resultado, payload, resposta e duração. | TRANSCRICAO | [09:34] Marcos |
| FDD-AC-17 | `docs/FDD.md` | Critério de Aceitação | O cliente deduplica as entregas repetidas pelo identificador único enviado em X-Event-Id. | TRANSCRICAO | [09:25] Diego |
| FDD-AC-22 | `docs/FDD.md` | Critério de Aceitação | URLs HTTP são recusadas e payloads acima de 64 KB geram erro em vez de serem truncados. | TRANSCRICAO | [09:24] Larissa |
| FDD-AC-23 | `docs/FDD.md` | Critério de Aceitação | O código de HMAC e geração de secret passa por revisão de segurança de pelo menos dois dias úteis antes do deploy. | TRANSCRICAO | [09:46] Sofia |
| FDD-CODE-01 | `docs/FDD.md` | Integração com Código | changeStatus executa a transação que atualiza o status e grava order_status_history; é o ponto proposto para publicação atômica. | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-CODE-02 | `docs/FDD.md` | Integração com Código | As transições válidas e os estados terminais do ciclo de vida de pedidos estão definidos em módulo próprio. | CODIGO | `src/modules/orders/order.status.ts` |
| FDD-CODE-03 | `docs/FDD.md` | Integração com Código | O middleware fornece authenticate e requireRole para aplicar a autorização prevista nas rotas da feature. | CODIGO | `src/middlewares/auth.middleware.ts` |

## Validação de Rastreabilidade

- **Quantidade total de itens identificados nos documentos:** 216 (RFC: 47; FDD: 55; ADRs: 66; PRD: 48).
- **Quantidade de itens rastreados:** 215.
- **Percentual de cobertura documental:** 215/216 = **99.54%**.
- **Quantidade total de linhas de dados no Tracker:** 215.
- **Linhas com Fonte = TRANSCRICAO:** 203 (94.42%).
- **Linhas com Fonte = CODIGO:** 12 (5.58%).
- **Cobertura específica do PRD:** 47/48 = **97.92%**; o item não rastreado é `PRD-AC-06`.
- **Evidência parcial no PRD:** `PRD-FR-02` comprova a consulta de endpoints, mas não a não exposição de secrets; `PRD-NFR-09` comprova o histórico e os tempos de resposta, mas não todo o conjunto de métricas, correlação e logging específico descrito.
- **Critério sem evidência:** `PRD-AC-06` define p95 inferior a 10 segundos para 100 eventos em homologação. A necessidade de entrega em menos de 10 segundos está na transcrição, mas o cenário p95/100 eventos não está demonstrado por transcrição ou código e, por isso, não recebeu linha própria.
- **Outros trechos não convertidos em linhas:** objetivos e itens “dentro do escopo” que repetem requisitos já identificados; riscos qualitativos explicitamente apresentados como inferências; dependências e passos de teste que não acrescentam requisito ou decisão independente com evidência primária.
- **Validação dos caminhos de código:** as 12 referências `CODIGO` foram previamente conferidas contra arquivos existentes em `src/`; as evidências correspondem a componentes existentes, não a implementação de webhooks.
- **Validação dos timestamps e falantes:** as 203 localizações `TRANSCRICAO` utilizam timestamps e falantes registrados literalmente em `TRANSCRICAO.md`.
- **Identificadores:** 215 IDs únicos, sem duplicatas.

### Resultado dos requisitos quantitativos

| Requisito | Resultado | Status |
|---|---:|---|
| Cobertura documental mínima de 80% | 99.54% | Atendido |
| Linhas com TRANSCRICAO mínimas de 70% | 94.42% | Atendido |
| Pelo menos 5 linhas com CODIGO | 12 | Atendido |
| Todos os caminhos de código existem | 12 caminhos verificados | Atendido |

**Nota de contagem:** o inventário considera itens atômicos identificáveis; resumos repetidos de requisitos em objetivos/escopo não são contados novamente. O critério `PRD-AC-06` permanece no denominador por ser um item explícito do PRD, embora não tenha evidência primária verificável e não possa ser incluído na tabela sem violar as regras de rastreabilidade.