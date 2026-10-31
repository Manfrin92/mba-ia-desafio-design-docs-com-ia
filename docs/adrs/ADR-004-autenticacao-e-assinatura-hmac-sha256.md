# ADR-004 — Autenticação e assinatura HMAC-SHA256

> Legenda de rastreabilidade: **[Reunião hh:mm]** = trecho da transcrição; **[Código]** = estado atual do repositório; **[Inferido]** = dedução do autor, não discutida na reunião.

## Status

**Aceito**, com condição: Sofia exige pelo menos dois dias úteis de revisão do código de segurança (HMAC e geração de secret) antes do deploy [09:46–09:49]. Decisão fechada em [09:22] e confirmada no resumo [09:48].

## Contexto

Os eventos contêm dados de pedidos e saem para endpoints fora da nossa infraestrutura. O cliente precisa validar a origem e a integridade da requisição [09:19]. Diego lembrou que já houve cliente que vazou secret em log da própria aplicação [09:22].

**[Código]**

- `authenticate` e `requireRole` (`src/middlewares/auth.middleware.ts`) tratam JWT com `AuthUser.role` igual a `'ADMIN'` ou `'OPERATOR'`.
- `src/shared/logger/index.ts` redige apenas `req.headers.authorization`, `req.headers.cookie`, `*.password`, `*.passwordHash`, `*.token` e `*.accessToken`.
- O modelo `User` em `prisma/schema.prisma` (linhas 25–38) não tem vínculo com `Customer`.

## Decisão

1. **HMAC-SHA256 sobre o corpo do request**, enviado no header `X-Signature` [09:20, 09:22].
2. **Secret única por endpoint**, nunca global ("se vaza uma, vaza tudo") [09:21]. A configuração do webhook guarda url, secret, `customer_id` e estado ativo [09:21].
3. **Geração da secret.** É feita pela plataforma e devolvida na criação [09:31].
4. **Rotação.** O cliente pede nova secret por endpoint da API. A antiga continua válida por 24 horas em paralelo e depois deixa de valer [09:21–09:22].
5. **Headers do envio** [09:44–09:45]:
   - `X-Event-Id` (ver ADR-005);
   - `X-Signature`;
   - `X-Timestamp` (timestamp do envio, para o cliente detectar replay se quiser);
   - `X-Webhook-Id` (id do endpoint);
   - `Content-Type: application/json`.
6. **Acesso à API de gestão.** Usa JWT do sistema, e qualquer role autenticada pode fazer o CRUD "por enquanto", com possibilidade de endurecer depois [09:36–09:37]. O `customer_id` vai no body ou no path, e não vem do JWT [09:32]. O replay de DLQ exige ADMIN (ADR-003).
7. **Registrado, mas fora desta decisão.** URL obrigatoriamente `https` (validação Zod, [09:23]) e limite de 64 KB com erro [09:24]. Foram tratados como validação e requisito não funcional, não como decisão arquitetural.

## Alternativas

- **Secret global da plataforma.** Rejeitada [09:21].
- **Rotação sem período de convivência.** Preterida pelo grace period de 24 horas [09:21].
- **[Inferido] mTLS, OAuth2 client credentials ou JWT assinado como autenticação do envio.** Não foram discutidos.
- **[Inferido] Assinatura que inclua o timestamp no conteúdo assinado.** Não foi discutida.

## Consequências

**Discutidas**

- O cliente pode verificar origem e integridade com bibliotecas comuns ([09:20]: "padrão de mercado").
- O vazamento de uma secret afeta um único endpoint.
- A rotação não exige parada do cliente.
- O ponto exige revisão de segurança antes do deploy.

**Deduzidas [Inferido]**

- Como o HMAC cobre só o corpo, o `X-Timestamp` não é protegido pela assinatura. A detecção de replay pelo cliente fica limitada.
- A secret precisa ser legível pelo worker para assinar, então não pode ser guardada só como hash.
- A redação de logs atual não cobre campos como secret ou assinatura. Dado o histórico de vazamento em log [09:22], seria preciso estender `redactPaths`.
- Não há dependência de assinatura em `package.json`. O módulo `node:crypto` cobre HMAC-SHA256.

**Pontos em aberto**

- Formato de `X-Signature` (hex, base64, prefixo).
- Como assinar durante o grace period, com duas secrets válidas: assinatura única ou dupla.
- Armazenamento da secret (texto claro ou criptografada) e se ela pode ser consultada depois da criação.
- **Divergência código × reunião.** Marcos disse que existem "usuários que representam o cliente" [09:32]. No código, `User` não tem relação com `Customer`. Com `customer_id` no body e CRUD aberto a qualquer role autenticada, não há controle sobre quem pode configurar webhooks de qual cliente. A reunião não tratou disso.
- Caminho do endpoint de rotação.
