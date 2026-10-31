# ADR-002 — Worker separado, polling e processamento sequencial

> Legenda de rastreabilidade: **[Reunião hh:mm]** = trecho da transcrição; **[Código]** = estado atual do repositório; **[Inferido]** = dedução do autor, não discutida na reunião.

## Status

**Aceito.** Polling de 2 segundos [09:10]. Processo separado [09:11]. Limitação de ordering registrada [09:13]. Resumo confirmado em [09:48–09:49].

## Contexto

O requisito de latência é menos de 10 segundos [09:02]. O MySQL não tem listener nativo como o NOTIFY/LISTEN do Postgres. Triggers só executam SQL e não avisam processos externos, e qualquer improviso "fica esquisito" [09:09]. Diego também apontou que, se o worker rodasse na instância da API, um restart da API o derrubaria [09:11].

**[Código]**

- `src/server.ts` é o único entry point. O `bootstrap()` monta o app, faz `listen`, trata SIGINT e SIGTERM e chama `prisma.$disconnect()`.
- Em `package.json`, os scripts `dev` e `start` apontam só para `server.ts` e `dist/server.js`. Não existe script de worker.
- `src/config/database.ts` exporta `prisma`, criado por `createPrismaClient()` a cada processo.
- `src/config/env.ts` exige `DATABASE_URL` e `JWT_SECRET` (mínimo de 16 caracteres) na importação.

## Decisão

1. **Processo separado.** O worker é um novo entry point `src/worker.ts` com script `npm run worker` [09:11]. A lógica de processamento fica no módulo, em `src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts` [09:28]. O nome exato não foi fechado.
2. **Banco e Prisma.** Usa o mesmo banco e a mesma `DATABASE_URL`, com um `PrismaClient` próprio, porque é outro processo Node [09:29–09:30].
3. **Polling a cada 2 segundos.** Busca os eventos pendentes mais antigos em lote pequeno, processa e marca [09:08–09:09]. Larissa registrou a latência de polling como 2 segundos no pior caso, e a equipe aceitou [09:10].
4. **Processamento sequencial.** É um único worker, que processa por ordem de `created_at` da outbox [09:12]. A ordenação é implícita por `order_id`, e só enquanto houver um worker. Larissa: "Documentamos como limitação conhecida. Não é garantia de ordering global" [09:13]. Marcos disse que os clientes nunca pediram ordering global [09:14].
5. **Timeout HTTP de 10 segundos.** Uma resposta que não chega nesse prazo é tratada como falha e marcada para retry [09:42]. A política de retry está na ADR-003.

## Alternativas

- **Chamada síncrona na API.** Ver ADR-001.
- **Trigger no banco para acionar o worker.** Rejeitada [09:09].
- **LISTEN/NOTIFY.** Inviável no MySQL [09:09].
- **Worker dentro do processo da API.** Rejeitada [09:11].
- **Múltiplos workers em paralelo.** Adiada. Particionar por `order_id` ou usar lock pessimista seriam os caminhos, mas "é problema do futuro" [09:13].
- **[Inferido] Cron ou job scheduler externo.** Não foi discutido.
- **[Inferido] Polling com intervalo adaptativo.** Não foi discutido.

## Consequências

**Discutidas**

- Sem infraestrutura adicional além de um processo Node.
- Latência de polling de até 2 segundos, aceita pelo PM [09:10].
- Ordering por `order_id` só com um único worker.
- Escalar para vários workers perde a garantia de ordering [09:12].

**Deduzidas [Inferido]**

- **Head-of-line blocking.** Como o processamento é sequencial e o timeout é de 10 segundos, um endpoint lento ou fora do ar atrasa os eventos de todos os outros. Isso pode comprometer a meta de menos de 10 segundos. O "atende tranquilo" de [09:09] vale para o caso normal, não como garantia absoluta.
- **Ordering e retry.** Com o backoff da ADR-003, um evento em espera de retry pode ser ultrapassado por um evento posterior do mesmo pedido. A reunião não discutiu isso.
- **Worker parado.** Os eventos continuam persistidos na outbox, mas acumulam sem entrega. Não há alta disponibilidade.
- **Lacunas operacionais.** Não foram definidos:
  - recuperação de linhas presas em "processando" após queda do worker;
  - tamanho do lote;
  - shutdown gracioso do worker (o padrão de `src/server.ts` é uma referência possível).
- **Configuração e logs.** Importar `src/config/env.ts` no worker exige `JWT_SECRET`, a menos que o schema seja separado. O logger (`src/shared/logger/index.ts`) fixa `service: 'order-management-api'`, então os logs do worker teriam o mesmo rótulo.
- **Escala futura.** MySQL 8.0 (imagem do `docker-compose.yml`) suporta `FOR UPDATE SKIP LOCKED`, que seria uma opção. Isso não foi dito na reunião.

**Pontos em aberto**

- Rate limiting de envio por cliente: Diego levantou o caso de 50 pedidos por minuto, e a decisão foi "observar e decidir depois" [09:38–09:39].
