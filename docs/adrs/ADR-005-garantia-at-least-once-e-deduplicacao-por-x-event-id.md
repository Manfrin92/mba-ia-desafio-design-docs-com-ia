# ADR-005 — Garantia de entrega at-least-once e deduplicação por X-Event-Id

> Legenda de rastreabilidade: **[Reunião hh:mm]** = trecho da transcrição; **[Código]** = estado atual do repositório; **[Inferido]** = dedução do autor, não discutida na reunião.

## Status

**Aceito.** Larissa registrou como decisão em [09:26], e o resumo [09:48] foi confirmado em [09:49].

## Contexto

Com retry (ADR-003) e timeout de 10 segundos (ADR-002), o mesmo evento pode chegar mais de uma vez ao cliente. Diego avaliou que exactly-once exigiria coordenação dos dois lados e seria "muito mais complexo" [09:25].

**[Código]** O projeto já usa UUIDs: `@default(uuid())` em `prisma/schema.prisma` e `v4 as uuidv4` do pacote `uuid` em `src/middlewares/request-logger.middleware.ts`. `uuid` 11.0.3 está em `package.json`.

## Decisão

1. **Garantia at-least-once.** O cliente pode receber o mesmo evento duas vezes e "tem que estar preparado" [09:24].
2. **Identificador de deduplicação.** O header `X-Event-Id` leva um UUID gerado quando o evento entra na outbox, único por evento [09:25].
3. **Deduplicação do lado do cliente.** A plataforma não deduplica. O cliente usa o `event_id` [09:25].
4. **Documentação.** Marcos documentará o comportamento de forma destacada no portal de desenvolvedores [09:26].
5. **Justificativa dada.** Diego citou Stripe e GitHub como prática de mercado e afirmou que cobre "99% dos casos" [09:25]. São afirmações dele, sem medição apresentada.

## Alternativas

- **Exactly-once.** Rejeitada por exigir coordenação dos dois lados [09:25].
- **[Inferido] At-most-once (sem retry).** Não foi discutida, e é incompatível com a ADR-003.
- **[Inferido] Deduplicação no servidor, com confirmação ou chave de idempotência negociada com o cliente.** Não foi discutida.

## Consequências

**Discutidas**

- A responsabilidade de deduplicar passa para o cliente. Sofia: "Isso joga responsabilidade pro cliente" [09:25].
- O design é mais simples do que exactly-once.

**Deduzidas [Inferido]**

- Duplicatas podem vir de:
  - timeout quando o cliente já processou;
  - queda do worker entre a resposta HTTP e a marcação como entregue;
  - retries;
  - replay de DLQ.
- A garantia não é absoluta. Depende de a transação ter commitado [09:06] e, esgotado o retry, o evento aguarda replay manual na DLQ.
- Duplicatas e retries se combinam com a limitação de ordering da ADR-002. O cliente pode receber eventos duplicados e fora de ordem.
- O cliente precisa guardar os `event_id` já vistos. A janela de retenção não foi definida. Ela deveria cobrir ao menos a janela de retry e o replay, que pode ocorrer muito depois.

**Pontos em aberto**

- Se o replay reutiliza o `event_id` original.
- Cardinalidade evento × endpoint (ver ADR-001). Se for uma linha por endpoint, o `event_id` seria distinto por endpoint, e o `X-Webhook-Id` diferenciaria os cadastros.
- Se o `event_id` é o mesmo UUID da linha da outbox.
