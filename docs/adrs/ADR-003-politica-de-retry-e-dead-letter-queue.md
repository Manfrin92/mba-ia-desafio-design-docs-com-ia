# ADR-003 — Política de retry e Dead Letter Queue (DLQ)

> Legenda de rastreabilidade: **[Reunião hh:mm]** = trecho da transcrição; **[Código]** = estado atual do repositório; **[Inferido]** = dedução do autor, não discutida na reunião.

## Status

**Aceito.** Retry com backoff exponencial e 5 tentativas [09:17]. DLQ em tabela separada [09:18]. Replay manual por endpoint admin [09:18–09:19]. Role ADMIN [09:36]. Resumo confirmado em [09:48–09:49].

## Contexto

Se o endpoint do cliente estiver offline, é preciso reenviar sem deixar o evento pendurado para sempre. Diego citou clientes com indisponibilidade de duas horas em manutenção planejada [09:16].

**[Código]** `requireRole` (`src/middlewares/auth.middleware.ts`) já existe e é usado com `requireRole('ADMIN')` em `src/modules/users/user.routes.ts`. O enum `UserRole` (`ADMIN`, `OPERATOR`) está em `prisma/schema.prisma`. O padrão de auditoria existente é `order_status_history`, com `changedById` e `reason`.

## Decisão

1. **Backoff exponencial com limite de 5 tentativas.** A progressão é 1 min, 5 min, 30 min, 2 h e 12 h [09:15–09:17]. Larissa: "Decidido: 5 tentativas, backoff 1m/5m/30m/2h/12h".
2. **Esgotadas as tentativas, a falha é permanente** e o evento vai para a DLQ [09:15].
3. **DLQ em tabela separada**, `webhook_dead_letter`, com payload, motivo da falha e timestamp. Fica mais limpa que marcar "failed" na outbox e serve de evidência para debug e reprocessamento [09:18].
4. **Replay manual** via `POST /admin/webhooks/dead-letter/:id/replay`, que recoloca o evento na outbox como pendente [09:18].
5. **Autorização do replay.** Exige role ADMIN, reaproveitando `requireRole`. O replay deve registrar quem o executou, para auditoria [09:35–09:36].
6. **Falha = erro de entrega ou timeout de 10 segundos** (ADR-002) [09:42].
7. **Fora de escopo:** aviso por e-mail ao cliente em caso de falhas, talvez numa fase futura [09:37–09:38].

## Alternativas

- **3 tentativas.** Rejeitada. Cobriria cerca de 30 minutos e mataria o evento numa indisponibilidade maior [09:16].
- **Retry indefinido com backoff.** Rejeitada. O evento ficaria pendurado se o cliente sumisse [09:15].
- **Marcar `failed` na própria outbox em vez de uma DLQ.** Rejeitada [09:18].
- **Replay aberto a qualquer role autenticada.** Rejeitada. Sofia: "não é coisa de operador" [09:36].
- **Notificação por e-mail.** Adiada [09:37–09:38].
- **[Inferido] Replay automático, ou pausa do endpoint após N falhas (circuit breaker).** Não foram discutidos.

## Consequências

**Discutidas**

- Cobertura de uma janela de indisponibilidade do cliente. Marcos considera aceitável que um cliente fora do ar por cerca de 15 horas tenha "problema sério" [09:17].
- DLQ preservada para debug.
- Reprocessamento depende de ação humana com perfil ADMIN.

**Deduzidas [Inferido]**

- Eventos na DLQ só são entregues após replay. A garantia de entrega da ADR-005 vale dentro da política de retry, não indefinidamente.
- O polling precisa respeitar o horário da próxima tentativa. Isso exige um campo de próxima tentativa e um contador de tentativas, que a reunião não nomeou.

**Pontos em aberto**

- **Contagem de tentativas.** "5 tentativas" pode significar 5 envios no total ou 1 envio inicial mais 5 retentativas. Os cinco intervalos somam 14h36, o que bate com "quase 15 horas" [09:17], e isso sugere 6 envios. Larissa falou em "total 5 tentativas" [09:48]. A reunião não resolveu a ambiguidade.
- **Janela de cobertura.** Diego mencionou "12 ou 24 horas" [09:15] e depois "quase 15" [09:17].
- **Critério de falha.** Não foi definido quais códigos HTTP contam como falha (por exemplo 4xx).
- **Jitter.** Não foi discutido.
- **Semântica do estado `falhou`.** [09:08] lista o estado "falhou" na outbox. Não está claro se ele é transitório (aguardando retry) ou equivale à falha permanente da DLQ.
- **Replay.** Não foi definido:
  - se o contador de tentativas é zerado;
  - se o `event_id` original é mantido (ver ADR-005);
  - se a linha sai da DLQ.
- **Auditoria.** "Logar quem fez o replay" não especifica log Pino ou registro persistido.
- **Caminho das rotas.** `src/routes/index.ts` monta todos os módulos em `/api/v1`. A reunião cita `/admin/webhooks/...` e `/webhooks/:id/deliveries` sem esse prefixo.
- **Histórico de entregas.** `GET /webhooks/:id/deliveries` (últimos 100, com payload, resposta e tempo de resposta [09:34]) exige registrar cada tentativa. Onde isso é armazenado não foi definido.
