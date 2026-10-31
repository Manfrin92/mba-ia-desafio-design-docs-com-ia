# ADR-001 — Transactional Outbox com MySQL

> Legenda de rastreabilidade: **[Reunião hh:mm]** = trecho da transcrição; **[Código]** = estado atual do repositório; **[Inferido]** = dedução do autor, não discutida na reunião.

## Status

**Aceito.** Larissa fechou a decisão em [09:08] e ela foi confirmada no resumo de [09:48], sem objeções (Diego, Bruno, Sofia e Marcos, [09:49]).

Duas decisões secundárias (UUID e snapshot do payload) foram tomadas em [09:51–09:52], depois que Marcos e Sofia já tinham saído da call. Estavam presentes apenas Larissa, Diego e Bruno.

Larissa ainda marcou uma sessão de revisão do design doc com Bruno e Diego antes de iniciar o código [09:50].

## Contexto

Três clientes B2B (Atlas Comercial, MaxDistribuição e Nova Cargo) pediram notificação quando o status dos pedidos muda. Hoje consultam `GET /orders` periodicamente [09:00]. Para eles, "tempo real" é menos de 10 segundos [09:02]. O escopo é só webhook de saída [09:02–09:03].

Bruno rejeitou o envio síncrono [09:04]. A transação de mudança de status já é pesada, e um cliente lento ou fora do ar travaria mudanças de outros pedidos. Reverter a mudança de status por falha do cliente também não faz sentido.

**[Código]** O fluxo está em `OrderService.changeStatus` (`src/modules/orders/order.service.ts`, linhas 126–179). Ele roda dentro de `this.prisma.$transaction(async (tx) => …)` e faz, em sequência:

- débito ou reposição de estoque, conforme `shouldDebitStock` e `shouldReplenishStock` (`src/modules/orders/order.status.ts`);
- `tx.order.update`;
- `tx.orderStatusHistory.create`.

O tipo `TxClient = Prisma.TransactionClient` já existe na linha 24 do mesmo arquivo. Não há nenhum mecanismo de eventos, filas ou webhooks em `src/`, e nenhuma ocorrência de `webhook`, `outbox` ou `worker`.

## Decisão

1. **Outbox no MySQL existente.** Dentro da mesma transação SQL que atualiza `orders` e `order_status_history`, o sistema insere uma linha numa tabela de outbox [09:06]. O nome citado foi "tipo `webhook_outbox`", ou seja, indicativo.
2. **Atomicidade.** Se a inserção na outbox falhar, a transação inteira sofre rollback. Não pode haver mudança de status sem evento [09:40]. Diego: "Se ficar fora da transação, perde a garantia toda" [09:41].
3. **Mecanismo de integração.** Bruno propôs a função `publishWebhookEvent(tx, order, fromStatus, toStatus)`, que recebe o client da transação corrente e é chamada por `order.service` [09:41]. Diego concordou ("função pura recebendo o tx. Não precisa injetar repository inteiro"). O nome e a assinatura vêm da proposta e ainda não existem no código.
4. **Estados e índices.** A tabela tem os estados pendente, processando, falhou e entregue, com índices em status e `created_at` [09:08].
5. **Filtro na inserção.** O webhook escolhe quais status quer receber. O filtro é aplicado na inserção: se nenhum webhook do customer quer aquele status, a linha nem é criada [09:33–09:34].
6. **Snapshot.** O payload já renderizado é gravado na inserção, para refletir o estado do pedido no momento da mudança [09:52]. O conteúdo definido em [09:43] é:
   - `event_id`;
   - `event_type` (`order.status_changed`);
   - `timestamp` ISO 8601;
   - `order_id`, `order_number`;
   - `from_status`, `to_status`;
   - `customer_id`;
   - campos básicos do pedido, como `total_cents`.

   Os `items` ficam de fora.
7. **Identificadores.** UUID, "segue o padrão do resto do projeto" [09:51]. **[Código]** O padrão existe em `prisma/schema.prisma`: `@id @default(uuid()) @db.Char(36)`.
8. **Arquivamento.** Arquivar linhas entregues após cerca de 30 dias está fora do escopo da feature [09:08].

## Alternativas

- **Chamada HTTP síncrona no `OrderService`.** Rejeitada [09:04–09:06]. Acopla a transação à disponibilidade do cliente.
- **Redis Streams ou similar.** Rejeitada [09:07]. Exigiria nova infraestrutura, e Diego a chamou de overengineering para um time pequeno.
- **Injetar um repository de webhook no `OrderService`.** Descartada em favor da função com `tx` [09:41].
- **Guardar só `order_id` e renderizar o payload no envio.** Rejeitada [09:51–09:52], porque produziria "caso esquisito" se o pedido mudasse depois.
- **ID auto incremental.** Preterido em favor de UUID [09:51].
- **[Inferido] Publicar o evento após o commit, fora da transação.** Não foi discutida como opção, mas [09:41] descreve o efeito: perde-se a garantia.
- **[Inferido] Usar `order_status_history` como fonte dos eventos.** A tabela é gravada atomicamente, mas não guarda estado de entrega, retry nem snapshot de payload. Não foi discutida.
- **[Inferido] CDC pelo binlog do MySQL.** Não foi discutida.

## Consequências

**Discutidas na reunião**

- Atomicidade entre pedido e evento, sem inconsistência possível quando o commit ocorre [09:06].
- Nenhuma infraestrutura nova.
- O filtro na inserção economiza linhas [09:34].

**Deduzidas [Inferido]**

- A transação de `changeStatus` ganha pelo menos mais um `INSERT`. Bruno já a considera pesada [09:04].
- Em `changeStatus`, `order` é lido (linha 132) antes do `update` (linha 158). O payload deve usar o argumento `to`, e não `order.status`.
- A outbox cresce no mesmo MySQL dos dados de negócio. O arquivamento ficou fora do escopo, então a tabela só cresce até que algo o implemente.
- `tests/setup.ts` limpa as tabelas com `deleteMany`. As novas tabelas precisarão entrar nessa limpeza, antes de `order`.

**Pontos em aberto (não decididos)**

- **Criação de pedido.** `OrderService.create` (linhas 95–120) grava histórico com `fromStatus: null` → `PENDING`. A reunião só tratou mudança de status, então não está decidido se a criação emite evento. Se emitisse, `from_status` seria nulo.
- **Cardinalidade.** Não está definido se há uma linha de outbox por evento ou por endpoint. O `X-Webhook-Id` [09:44–09:45] sugere envio por endpoint. O `event_id` é descrito como "único por evento" [09:25].
- **Relação com `orders`.** `OrderService.delete` (linhas 181–192) permite apagar pedidos `PENDING` ou `CANCELLED`, e o schema usa `onDelete: Cascade` no histórico. O comportamento esperado para linhas de outbox desse pedido não foi discutido.
- **Nomenclatura.** A transcrição fala em `created_at` e `stock_quantity`. No schema, as colunas são camelCase (`createdAt`, `stockQuantity`) e só as tabelas são snake_case via `@@map`. A convenção da nova tabela fica para a modelagem.
- **`event_id` e id da linha.** Não está dito se são o mesmo UUID.
