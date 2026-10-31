# ADR-006 — Reutilização dos padrões arquiteturais existentes

> Legenda de rastreabilidade: **[Reunião hh:mm]** = trecho da transcrição; **[Código]** = estado atual do repositório; **[Inferido]** = dedução do autor, não discutida na reunião.

## Status

**Aceito.** Larissa: "Decisão: reuso máximo do que já existe" [09:30]. Resumo confirmado em [09:48–09:49].

## Contexto

Bruno apontou que a codebase tem um padrão claro de módulos em `src/modules` e propôs seguir o mesmo para webhooks [09:27]. A reunião listou os padrões a reaproveitar [09:28–09:30], e o resumo repetiu a lista [09:48].

## Decisão

| Padrão | Evidência no código | Uso em webhooks |
|---|---|---|
| Módulo por domínio com controller, service, repository, routes e schemas | `src/modules/orders/` (`order.controller.ts`, `order.service.ts`, `order.repository.ts`, `order.routes.ts`, `order.schemas.ts`) | Novo `src/modules/webhooks` [09:27–09:28] |
| Entry points | `src/server.ts` | Novo `src/worker.ts` (ADR-002) |
| `AppError` e subclasses | `src/shared/errors/app-error.ts` e `http-errors.ts` | Erros do módulo com código prefixado `WEBHOOK_`, por exemplo `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED` "etc." [09:28–09:29] |
| Logger Pino | `src/shared/logger/index.ts` (`logger`) | Sem biblioteca nova [09:29] |
| Middleware de erro centralizado | `src/middlewares/error.middleware.ts` (trata `AppError`, `ZodError` e `Prisma` P2002 e P2025) | Reuso nas rotas do módulo [09:29] |
| Schemas Zod e validação | `src/modules/orders/order.schemas.ts`, `src/middlewares/validate.middleware.ts` | Mesmo padrão [09:30] |
| Autenticação e roles | `authenticate` e `requireRole` em `src/middlewares/auth.middleware.ts` | ADMIN no replay [09:36] |
| Prisma, MySQL e UUID | `prisma/schema.prisma`, `src/config/database.ts` | Mesmo banco, `PrismaClient` por processo [09:30, 09:51] |
| Transação | `OrderService.changeStatus` e `TxClient` em `order.service.ts` | `publishWebhookEvent(tx, …)` (ADR-001) |

## Alternativas

- **Infraestrutura ou bibliotecas novas** (logger, fila, tratamento de erro próprio). Rejeitadas de forma implícita: "Não vamos botar nada novo" [09:29].
- **[Inferido] Colocar a lógica de webhooks dentro do módulo `orders`, ou em um serviço separado.** Não foram discutidas.

## Consequências

**Discutidas**

- Consistência com o restante da codebase.
- Menos código novo e menos infraestrutura.

**Deduzidas [Inferido]**

- **Registro do módulo.** Exige mudanças em `src/app.ts` (`buildControllers`, onde a injeção de dependências é manual) e em `src/routes/index.ts` (`Controllers` e `buildApiRouter`).
- **Acoplamento.** `OrderService` passa a depender do módulo de webhooks, via `publishWebhookEvent`.
- **Testes.** `tests/setup.ts` precisará limpar as novas tabelas.

**Limites e divergências**

- **Códigos de erro.** Os códigos atuais não têm prefixo de módulo (`INSUFFICIENT_STOCK`, `INVALID_STATUS_TRANSITION`). O prefixo `WEBHOOK_` é uma convenção nova, ainda que coerente com o padrão de ter um código por erro.
- **`NotFoundError` e `ValidationError`.** Em `http-errors.ts`, eles fixam os códigos `NOT_FOUND` e `VALIDATION_ERROR` e não aceitam código customizado. `WEBHOOK_NOT_FOUND` exigiria uma nova subclasse ou a alteração dessas classes. A reunião não tratou disso. `BadRequestError`, `ConflictError` e `UnprocessableEntityError` já aceitam código.
- **Middleware de erro.** Bruno disse que ele pegaria os erros "sem precisar mudar nada" [09:29]. Isso vale para as rotas HTTP. O worker não passa pelo pipeline do Express, então precisará de tratamento e logging próprios. A reunião não definiu isso.
- **Lista de códigos.** A lista de `WEBHOOK_*` não está fechada.
- **Prefixo de rota.** As rotas deveriam entrar sob `/api/v1`, como as demais, mas a reunião citou caminhos sem esse prefixo.
