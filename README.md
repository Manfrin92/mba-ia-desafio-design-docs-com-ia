# Sobre o desafio nas minhas palavras

O desafio era gerar documentação para que desenvolvedores ou agentes de IA implementassem uma ou várias features novas necessárias para atender novos requisitos de clientes. A ideia foi que a documentação em si também seria gerada com IA e isso seria feito a partir de uma transcrição de uma reunião com as pessoas que discutiram a nova feature do ponto de vista de cliente e técnico. Então, com a transcrição bem definida era possível pegar o que foi decidido e comparando com o código fonte existente criar toda documentação que poderia ser usada pelo time que de fato faria a entrega daquela nova funcionalidade. Na vida real eu vejo que essa transcrição ainda pode ser um pouco complicada de se obter, tenho usado o Zoom para resumos similares, mas é bem claro que muita coisa fica super confusa e ele não parece entender bem o que foi dito, da minha experiência prática isso acontece mais em reuniões em inglês que são os idiomas que geralmente trabalho e gravamos esse tipo de resumo. No futuro acho que isso pode evoluir e aí sim termos algo fidedigno que possa ser utilizado da mesma maneira que nesse trabalho. Talvez o que temos hoje possa ser utilizado como ponto de partida para um posterior refinamento antes do envio final ao time que faz a entrega de fato. De qualquer maneira, a possibilidade de conseguir juntar o contexto inteiro do código com reuniões em que se discutem novas funcionalidades em poucos segundos é impressionante, até então sempre foi preciso ter pessoas com esse tipo de contexto em mente participando das reuniões para dar opiniões, mas agora com a IA é possível que isso mude um pouco talvez o próprio agente participando da reunião trazendo os pontos do código que são iguais ou diferente do que se está falando. Outro ponto que me chamou a atenção foi que talvez isso não seja assim tão viável em projetos grandes, projetos em que trabalhei sim seria possível pois praticamente todos são projetos pequenos e que atendem poucos usuários, mas penso que para empresas maiores isso pode acabar sendo um impeditivo - ou seria preciso delimitar a IA para módulos ou partes do código para ter respostas corretas.

# Ferramentas de IA utilizadas

Chat GPT 6, Claude Sonnet 5.5 tudo direto no chat sem agente modificando nada localmente na minha máquina. Usei o primeiro para refinar os prompts que eventualmente mandaria para o segundo, O GPT me ajudou a melhorar o prompt em si e ir entendendo aos poucos o que faltou, tentei fazer da melhor maneira que pensei sem consultas para treinar meu entendimento do que é um bom prompt logo no começo, mas poderia ter usado ele direto desde o início, mas acredito que isso não me ajudaria a treinar nada.

# Workflow adotado

Comecei pelas ADRs. A organização foi conforme explicado na sessão **Ferramentas de IA utilizadas**.

# Prompts customizados

## ADR
---
v0

Pretendo mandar um prompt para um agente gerar uma ADR em um formato e com um contéudo muito bem determinado, analise o prompt abaixo e me diga se algo assim é suficiente ou se devo ser mais explícito em alguma coisa.


<prompt-para-geracao-adr>
Estou realizando a documentacao para uma feature com base em uma reuniao técnica para um projeto que é um Order Management System (Node.js + TypeScript) URL https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia/tree/main/src com módulos de autenticação, usuários, clientes, produtos e pedidos. Banco MySQL via Prisma. O ciclo de vida do pedido tem máquina de estados controlada, controle transacional de estoque e auditoria de mudanças de status. A aplicação não tem nenhum mecanismo de notificação externa, eventos, filas ou webhooks, criar isso é o que foi discutido na reunião que tem a transcrição em anexo. Leia o código e a transcrição e crie,
<pedido>
1. Entre 5 e 8 ADRs
2. Cada ADR contém Status, Contexto, Decisão, Alternativas Consideradas (pelo menos 1 alternativa real discutida ou plausível), Consequências (positivas e negativas, com trade-off explícito)
3. NÃO INVENTE NADA DE FORA DA DISCUSSÃO OU DO CÓDIGO EXISTENTE
</pedido>
<requisitos>
1. Pelo menos 1 ADR deve referenciar explicitamente arquivos, módulos ou padrões do código existente
2. Pelo menos 1 ADR referencia explicitamente arquivos, módulos ou classes do código base
3. O conjunto de ADRs deve cobrir, no mínimo, 5 das 6 decisões principais discutidas na reunião:
a. Padrão Outbox no MySQL
b. Política de retry com backoff e DLQ
c. Autenticação HMAC-SHA256 com secret por endpoint
d. Garantia at-least-once com X-Event-Id
e. Worker em processo separado em polling
f. Reuso dos padrões existentes do projeto
g. Decisões técnicas secundárias não precisam aparecer das ADRs
</requisitos>
</prompt-para-geracao-adr>

---
v1

Fiz quase todas modificacoes sugeridas, veja agora com a transcricao em anexo


<prompt-para-geracao-adr>
Estou realizando a documentacao para uma feature com base em uma reuniao técnica para um projeto que é um Order Management System (Node.js + TypeScript) URL https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia/tree/main/src com módulos de autenticação, usuários, clientes, produtos e pedidos. Banco MySQL via Prisma. O ciclo de vida do pedido tem máquina de estados controlada, controle transacional de estoque e auditoria de mudanças de status. A aplicação não tem nenhum mecanismo de notificação externa, eventos, filas ou webhooks, criar isso é o que foi discutido na reunião que tem a transcrição em anexo. Leia o código e a transcrição e crie,
<pedido>
1. Entre 5 e 8 ADRs
2. Cada ADR contém,
Seção	O que deve conter
Status	Proposto, Aceito, Rejeitado ou Depreciado, conforme evidências
Contexto	Problema arquitetural e motivação da decisão
Decisão	O que será implementado e como funcionará
Alternativas	Opções consideradas e motivos para não escolhê-las
Consequências	Benefícios, limitações, custos e trade-offs
3. IMPORTANTE - Utilize exclusivamente informações presentes na transcrição e no código existente. Não apresente suposições como fatos ou decisões tomadas. Alternativas não discutidas explicitamente podem ser incluídas quando forem tecnicamente plausíveis, mas devem ser identificadas como alternativas inferidas pelo agente. Consequências também podem ser deduzidas tecnicamente, desde que sejam claramente distinguíveis dos pontos explicitamente discutidos na reunião.
</pedido>
<requisitos>
1. Pelo menos duas ADRs devem apresentar referências concretas ao código existente, incluindo caminhos de arquivos, módulos, classes ou funções. Essas referências devem ser relevantes para a decisão documentada, não apenas citações decorativas.
2. O conjunto de ADRs deve cobrir, no mínimo, 5 das 6 decisões principais discutidas na reunião:
a. Padrão Outbox no MySQL
b. Política de retry com backoff e DLQ
c. Autenticação HMAC-SHA256 com secret por endpoint
d. Garantia at-least-once com X-Event-Id
e. Worker em processo separado em polling
f. Reuso dos padrões existentes do projeto
g. Decisões técnicas secundárias não precisam aparecer das ADRs
3. Cada ADR deve representar uma decisão arquitetural independente, com escopo claramente delimitado. Não fragmente uma mesma decisão em múltiplas ADRs apenas para atingir a quantidade solicitada. Evite também agrupar decisões independentes que possam ser avaliadas e modificadas separadamente.
4. Sempre que uma ADR mencionar comportamento, implementação ou padrão existente, referencie os arquivos, classes, funções ou módulos correspondentes. Não invente caminhos ou nomes de classes. Quando não conseguir comprovar uma referência no código, indique essa limitação.
5. A transcrição representa as decisões discutidas na reunião. O código representa o estado atual da implementação. Não confunda decisões futuras com funcionalidades já existentes. Se houver divergências, identifique-as em vez de tentar resolvê-las por conta própria.
6. Antes de finalizar, verifique se todas as ADRs possuem as seções obrigatórias, se não existem decisões contraditórias entre documentos e se todas as referências ao código realmente existem.
</requisitos>
</prompt-para-geracao-adr>

---

v2

# Geração de ADRs — Sistema de Webhooks de Notificação de Pedidos

Estou realizando a documentação de uma feature com base em uma reunião técnica para um projeto que é um Order Management System (Node.js + TypeScript).

**Repositório:** https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia

Confirme se tem acesso ao repositório antes de continuar.

O projeto possui módulos de autenticação, usuários, clientes, produtos e pedidos. Utiliza MySQL via Prisma. O ciclo de vida do pedido possui máquina de estados controlada, controle transacional de estoque e auditoria de mudanças de status.

Atualmente, a aplicação não possui mecanismos de notificação externa, eventos, filas ou webhooks. A implementação desse mecanismo foi o tema da reunião técnica cuja transcrição está anexada a esta conversa.

Leia integralmente a transcrição e inspecione o código existente no repositório antes de produzir as ADRs.

<pedido>

1. Produza entre 5 e 8 ADRs.

2. Cada ADR deve conter obrigatoriamente as seguintes seções:

| Seção | O que deve conter |
|---|---|
| Status | Proposto, Aceito, Rejeitado ou Depreciado, conforme as evidências da reunião. |
| Contexto | Problema arquitetural e motivação da decisão. |
| Decisão | O que será implementado e como funcionará. |
| Alternativas | Opções consideradas e motivos para não escolhê-las. |
| Consequências | Benefícios, limitações, custos e trade-offs explícitos. |

3. Utilize exclusivamente informações presentes na transcrição e no código existente. Não apresente suposições como fatos ou decisões tomadas.

Alternativas não discutidas explicitamente podem ser incluídas quando forem tecnicamente plausíveis, mas devem ser identificadas como alternativas inferidas pelo agente.

Consequências também podem ser deduzidas tecnicamente, desde que sejam coerentes com a decisão e claramente distinguíveis dos pontos explicitamente discutidos na reunião.

</pedido>

<requisitos>

1. Pelo menos duas ADRs devem apresentar referências concretas ao código existente, incluindo caminhos de arquivos, módulos, classes ou funções. Essas referências devem ser relevantes para a decisão documentada, não apenas citações decorativas.

2. O conjunto de ADRs deve cobrir, no mínimo, 5 das 6 decisões principais discutidas na reunião:

   a. Padrão Outbox no MySQL.

   b. Política de retry com backoff e DLQ.

   c. Autenticação HMAC-SHA256 com secret por endpoint.

   d. Garantia at-least-once com X-Event-Id.

   e. Worker em processo separado utilizando polling.

   f. Reutilização dos padrões existentes do projeto.

   g. Decisões técnicas secundárias não precisam necessariamente aparecer em ADRs independentes.

3. Cada ADR deve representar uma decisão arquitetural independente, com escopo claramente delimitado. Não fragmente uma mesma decisão em múltiplas ADRs apenas para atingir a quantidade solicitada. Evite também agrupar decisões independentes que possam ser avaliadas e modificadas separadamente.

4. Sempre que uma ADR mencionar comportamento, implementação ou padrão existente, referencie os arquivos, classes, funções ou módulos correspondentes. Não invente caminhos ou nomes de classes. Quando não conseguir comprovar uma referência no código, indique essa limitação.

5. A transcrição representa as decisões discutidas na reunião. O código representa o estado atual da implementação. Não confunda decisões futuras com funcionalidades já existentes. Se houver divergências, identifique-as em vez de tentar resolvê-las por conta própria.

6. Preserve as limitações, ressalvas e pontos em aberto mencionados pelos participantes. Não transforme comportamentos esperados sob determinadas condições em garantias arquiteturais absolutas.

7. Não transforme sugestões em decisões aprovadas sem evidências na transcrição. Diferencie claramente decisões aprovadas, alternativas rejeitadas e assuntos deixados para fases futuras.

8. Considere decisões secundárias relacionadas às decisões principais quando forem relevantes para documentá-las. Por exemplo:
   - Atomicidade entre alteração do pedido e inserção no Outbox.
   - Snapshot do payload no momento da inserção.
   - UUID para identificação dos registros.
   - DLQ em tabela separada e replay manual.
   - Timeout HTTP do worker.
   - Ordering e limitações do processamento sequencial.

   Não crie ADRs independentes para decisões secundárias apenas para aumentar a quantidade de documentos.

9. Sempre que possível, identifique o horário da reunião em que a decisão foi discutida, facilitando a rastreabilidade entre a ADR e a transcrição.

10. Antes de finalizar, verifique se todas as ADRs possuem as seções obrigatórias, se não existem decisões contraditórias entre documentos e se todas as referências ao código realmente existem.

</requisitos>

<formato-de-saida>

1. Idioma: português brasileiro.
2. Formato: Markdown.
3. Produza cada ADR como um documento independente.
4. Utilize numeração sequencial: ADR-001, ADR-002, ADR-003 etc.
5. Utilize títulos descritivos para cada ADR.
6. Apresente cada ADR separadamente, com seu identificador e conteúdo completo.
7. Não crie arquivos no repositório, não altere o código e não tente realizar commits.
8. Não gere arquivos para download. Quero receber o conteúdo integral das ADRs diretamente na resposta, para que eu mesmo possa copiar e criar os arquivos posteriormente.
9. Não inclua explicações extensas sobre o processo de elaboração. Priorize a entrega das ADRs.

</formato-de-saida>

<analise-e-rastreabilidade>

Antes de escrever as ADRs:

1. Identifique todas as decisões arquiteturais presentes na transcrição.
2. Agrupe decisões relacionadas quando fizer sentido, respeitando a independência arquitetural de cada ADR.
3. Inspecione a estrutura completa do repositório, incluindo src, configurações, schema do Prisma, migrations, package.json e demais arquivos relevantes. Não limite a análise ao diretório src.
4. Identifique os padrões arquiteturais e de implementação já existentes que possam ser relevantes para as decisões discutidas.
5. Verifique se as referências ao código realmente existem.

Se não conseguir acessar o GitHub ou algum arquivo necessário, informe claramente a limitação. Não invente referências nem simule uma inspeção que não foi realizada.

A transcrição é a fonte de verdade para as decisões discutidas na reunião. O código é a fonte de verdade para o estado atual da implementação.

Se houver informações insuficientes para documentar alguma decisão, deixe isso explícito em vez de preencher lacunas com suposições.

</analise-e-rastreabilidade></prompt-para-geracao-adr>

---
v3

# Geração de ADRs — Sistema de Webhooks de Notificação de Pedidos

## Contexto

Você deve produzir documentação arquitetural (ADRs) para a implementação de um sistema de webhooks de notificação de pedidos.

**Repositório:** https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia

O projeto é um Order Management System desenvolvido em Node.js + TypeScript, com MySQL via Prisma. Possui módulos de autenticação, usuários, clientes, produtos e pedidos. O ciclo de vida dos pedidos utiliza uma máquina de estados controlada, controle transacional de estoque e auditoria de mudanças de status.

Atualmente, a aplicação não possui mecanismos de notificação externa, eventos, filas ou webhooks.

A funcionalidade foi discutida em uma reunião técnica, cuja transcrição está anexada a esta conversa.

Antes de produzir os ADRs, leia integralmente a transcrição e inspecione o repositório para compreender a implementação atual e seus padrões arquiteturais.

## Pedido

Produza exatamente **6 ADRs**, seguindo esta divisão:

1. **ADR-001 — Transactional Outbox com MySQL**
2. **ADR-002 — Worker separado, polling e processamento sequencial**
3. **ADR-003 — Política de retry e Dead Letter Queue (DLQ)**
4. **ADR-004 — Autenticação e assinatura HMAC-SHA256**
5. **ADR-005 — Garantia de entrega at-least-once e deduplicação por X-Event-Id**
6. **ADR-006 — Reutilização dos padrões arquiteturais existentes**

Cada ADR deve documentar uma decisão arquitetural coerente e independente. Decisões secundárias relacionadas devem ser incorporadas ao ADR correspondente, sem criar documentos adicionais ou fragmentar artificialmente as decisões.

## Estrutura obrigatória de cada ADR

Cada documento deve conter as seguintes seções:

- **Status:** Proposto, Aceito, Rejeitado ou Depreciado, conforme as evidências da reunião.
- **Contexto:** problema arquitetural, motivação e situação atual.
- **Decisão:** o que será implementado e como.
- **Alternativas:** opções consideradas e motivos para não selecioná-las.
- **Consequências:** benefícios, limitações, custos e trade-offs explícitos.

## Requisitos de conteúdo e rastreabilidade

1. Pelo menos dois ADRs devem conter referências concretas ao código existente, incluindo caminhos de arquivos, módulos, classes ou funções relevantes.
2. Os seis ADRs devem cobrir:
   - Transactional Outbox utilizando MySQL.
   - Worker em processo separado, utilizando polling.
   - Retry exponencial com backoff e DLQ.
   - HMAC-SHA256 com segredo individual por endpoint.
   - Entrega at-least-once e deduplicação via X-Event-Id.
   - Reutilização dos padrões arquiteturais existentes.
3. Cada ADR deve representar uma decisão arquitetural independente.
4. Sempre que mencionar um comportamento, implementação ou padrão existente, indique os arquivos, classes ou funções correspondentes. Nunca invente caminhos ou nomes. Se não for possível verificar uma referência, informe a limitação.
5. A transcrição é a fonte de verdade para as decisões discutidas na reunião. O código é a fonte de verdade para a implementação atual.
6. Não confunda decisões futuras com funcionalidades já implementadas.
7. Se houver divergência entre a transcrição e o código, identifique-a sem resolvê-la por conta própria.
8. Preserve limitações, ressalvas e questões em aberto. Não transforme comportamentos esperados sob determinadas condições em garantias absolutas.
9. Não apresente sugestões como decisões aprovadas sem evidências.
10. Diferencie decisões aprovadas, alternativas rejeitadas e assuntos adiados para fases futuras.
11. Incorpore decisões secundárias relevantes nos ADRs correspondentes, incluindo:
    - Atomicidade entre atualização do status do pedido e inserção no Outbox.
    - Snapshot do payload no momento da inserção.
    - Utilização de UUID.
    - Tabela separada para DLQ e replay manual.
    - Timeout HTTP do worker.
    - Limitações de ordenação e processamento sequencial.
12. Sempre que possível, identifique o timestamp da reunião que fundamenta cada decisão.
13. Antes de finalizar, verifique se os seis ADRs estão completos, coerentes entre si e sem contradições.
14. Verifique se todas as referências ao código realmente existem.

## Análise e preparação

Antes de escrever os documentos:

1. Identifique todas as decisões arquiteturais presentes na transcrição.
2. Agrupe decisões relacionadas, respeitando a independência de cada ADR.
3. Inspecione o repositório completo, incluindo `src`, configurações, schema do Prisma, migrations, `package.json` e demais arquivos relevantes.
4. Identifique os padrões arquiteturais e de implementação existentes.
5. Verifique se as referências ao código realmente existem.
6. Se não conseguir acessar o GitHub ou algum arquivo necessário, informe claramente a limitação. Não invente referências nem afirme ter inspecionado algo que não conseguiu acessar.

## Formato dos arquivos e entrega

Gere exatamente seis arquivos Markdown individuais, um para cada ADR.

Os nomes devem seguir obrigatoriamente o padrão:

`ADR-NNN-titulo-em-kebab-case.md`

Utilize os seguintes nomes:

- `ADR-001-transactional-outbox-com-mysql.md`
- `ADR-002-worker-separado-polling-processamento-sequencial.md`
- `ADR-003-politica-retry-dead-letter-queue.md`
- `ADR-004-autenticacao-assinatura-hmac-sha256.md`
- `ADR-005-entrega-at-least-once-deduplicacao-event-id.md`
- `ADR-006-reutilizacao-padroes-arquiteturais.md`

### Regras de entrega

1. Disponibilize os seis arquivos para download.
2. Apresente um link de download individual para cada arquivo.
3. Além dos links individuais, gere um arquivo ZIP contendo os seis ADRs e disponibilize também seu link de download.
4. Não substitua os arquivos por uma resposta contendo apenas o conteúdo Markdown no chat.
5. Não crie arquivos dentro do repositório original.
6. Não modifique código, configurações ou arquivos existentes no repositório.
7. Não faça commits ou alterações remotas no GitHub.
8. Os arquivos devem conter o conteúdo completo dos ADRs, não apenas resumos ou placeholders.
9. Utilize português brasileiro e Markdown.
10. Não inclua explicações extensas sobre o processo. Priorize a entrega dos arquivos e seus links.

---

## RFC

---
v0
Me ajude a refinar o prompt abaixo para que eu envie para outra llm possivelmente o Claude Sonnet 5.5 e tenha um arquivo de RFC conforme os requisitos abaixo. Me diga se da maneira como estou escrevendo é suficiente ou se preciso refinar mais esse prompt, minha ideia é copiar ele literalmente e simplesmente enviar em outro modelo em uma conversa do zero dando acesso apenas as ADRs em formato de markdown.

<prompt>
Com base em 6 ADRs em anexo gerados a partir de uma transcrição de uma reunião em do time técnico e time de produto, gere uma RFC com os requisitos abaixo,


Proposta técnica da solução para revisão: abordagem geral, alternativas e questões em aberto	Arquitetura	Como pretendemos resolver, e o que ainda está em aberto?

Produza o arquivo docs/RFC.md com a proposta técnica da solução, no formato de um documento submetido à equipe para revisão. O RFC opera em nível de arquitetura: apresenta a abordagem escolhida, as alternativas que foram colocadas na mesa e as questões deixadas em aberto. É um documento conciso (2 a 4 páginas); o detalhamento de implementação fica no FDD. Deve incluir no mínimo:

Metadados (autor, status, data, revisores); use os participantes da reunião como revisores
Resumo executivo (TL;DR) da proposta
Contexto e problema
Proposta técnica (visão geral da solução, sem descer ao detalhe de implementação do FDD)
Alternativas consideradas (pelo menos 2 alternativas reais discutidas e descartadas na reunião, cada uma com o trade-off que levou ao descarte)
Questões em aberto (pelo menos 2 pontos levantados na reunião e não decididos ou adiados)
Impacto e riscos
Decisões relacionadas (links para os ADRs correspondentes)
O RFC não deve duplicar o detalhamento do FDD. Ele responde "o que propomos e por quê"; o "como construir" em detalhe fica no FDD.

☐ Seção "Questões em aberto" lista pelo menos 2 pontos adiados ou não decididos na reunião
☐ Referencia, com link, pelo menos 2 ADRs do pacote

</prompt>

---
v1

# Geração de RFC — Proposta Técnica de Arquitetura

Com base exclusivamente nas ADRs em formato Markdown anexadas a esta conversa, produza um documento RFC (Request for Comments) contendo a proposta técnica consolidada da solução.

As ADRs foram elaboradas a partir de uma transcrição de reunião entre os times técnico e de produto. Elas representam as decisões arquiteturais tomadas, as alternativas discutidas e os pontos que permaneceram em aberto.

**Não utilize fontes externas, conhecimento geral ou suposições para complementar as informações das ADRs.** O objetivo é consolidar o conteúdo existente, não propor uma nova solução.

## 1. Objetivo do documento

Produzir `docs/RFC.md`, um documento submetido à equipe para revisão arquitetural.

O RFC deve responder principalmente:

- O que pretendemos construir?
- Qual abordagem técnica estamos propondo?
- Por que essa abordagem foi escolhida?
- Quais alternativas foram consideradas e descartadas?
- Quais decisões ainda precisam ser tomadas?
- Quais são os principais impactos e riscos da proposta?

O RFC deve operar no nível de arquitetura. Não deve reproduzir o detalhamento de implementação, que pertence ao FDD (Functional Design Document).

## 2. Regras de utilização das fontes

Siga rigorosamente estas regras:

1. Utilize todas as ADRs disponibilizadas como fontes do documento.
2. Considere as decisões registradas nas ADRs como a fonte de verdade para a proposta arquitetural.
3. Preserve a distinção entre informações discutidas na reunião, informações identificadas no código e inferências dos autores das ADRs.
4. Não apresente informações classificadas como `[Inferido]` como decisões tomadas ou alternativas efetivamente discutidas na reunião.
5. Não invente decisões, participantes, aprovações, requisitos, justificativas ou informações ausentes nas fontes.
6. Quando houver ambiguidades, divergências ou dependências não resolvidas entre ADRs, preserve-as explicitamente como questões em aberto.
7. Não tente resolver contradições por conta própria. O papel do RFC é expô-las para revisão da equipe.
8. Não introduza recomendações técnicas próprias ou alternativas que não estejam documentadas nas ADRs.

Se alguma ADR mencionada por outra ADR não estiver entre os arquivos disponibilizados, não tente reconstruir seu conteúdo. Registre a ausência e não crie referências fictícias.

## 3. Estrutura obrigatória do RFC

O documento deve conter as seguintes seções:

### 3.1. Metadados

- Título do RFC.
- Autor: utilize "Não informado nas fontes" caso não seja possível identificar.
- Status: Proposto para revisão.
- Data: data atual de geração, caso disponível.
- Revisores: utilize os participantes da reunião identificados nas ADRs.

Não confunda participação na reunião com aprovação formal do documento.

### 3.2. Resumo executivo (TL;DR)

Apresente uma síntese objetiva da proposta, incluindo:

- Problema que motivou a iniciativa.
- Abordagem arquitetural escolhida.
- Principais decisões que sustentam a solução.
- Principais questões ainda pendentes.

Um leitor deve conseguir entender a proposta sem precisar ler o documento inteiro.

### 3.3. Contexto e problema

Descreva:

- Cenário atual.
- Necessidade de negócio que motivou a iniciativa.
- Problema que a solução pretende resolver.
- Restrições e requisitos relevantes identificados nas reuniões.

Evite reproduzir detalhes do código existente, exceto quando forem necessários para explicar uma restrição arquitetural.

### 3.4. Proposta técnica

Apresente uma visão consolidada da arquitetura proposta.

Explique os principais componentes, suas responsabilidades e como interagem para atender aos requisitos.

Consolide as decisões das ADRs relacionadas, apresentando uma narrativa arquitetural coerente.

Inclua somente informações necessárias para compreender a solução e suas principais características.

Não entre em detalhes de implementação, como:
- Estrutura de diretórios e arquivos.
- Assinaturas de métodos.
- Modelagem detalhada de tabelas e colunas.
- Estruturas específicas de classes.
- Implementação de endpoints.
- Pseudocódigo ou trechos de código.

Esses detalhes pertencem ao FDD.

### 3.5. Alternativas consideradas

Apresente pelo menos duas alternativas reais que foram discutidas na reunião e posteriormente descartadas ou preteridas.

Para cada alternativa, descreva:
- Qual era a abordagem considerada.
- Por que foi considerada.
- Qual trade-off ou limitação levou à sua rejeição.
- Qual ADR registra essa discussão.

**Restrição importante:** utilize somente alternativas explicitamente discutidas nas reuniões e registradas nas ADRs. Não transforme alternativas classificadas como `[Inferido]` em alternativas discutidas.

Priorize alternativas que representem decisões arquiteturais relevantes, em vez de escolhas menores de implementação.

### 3.6. Questões em aberto

Apresente pelo menos duas questões que foram levantadas na reunião, mas não tiveram uma decisão definitiva ou foram explicitamente adiadas.

Para cada questão, apresente:
- Descrição do ponto pendente.
- Contexto ou motivo pelo qual permanece em aberto.
- ADR de origem, quando identificável.

Priorize questões que possam afetar a arquitetura, o comportamento da solução, a segurança ou decisões importantes de produto.

Não responda às questões por conta própria.

Questões identificadas posteriormente pelos autores das ADRs como inferências podem ser incluídas somente quando forem relevantes para a revisão arquitetural, deixando explícito que são pontos identificados a partir da análise, e não decisões ou discussões registradas na reunião.

### 3.7. Impactos e riscos

Consolide os principais impactos e riscos decorrentes da arquitetura proposta.

Considere aspectos como:
- Complexidade arquitetural.
- Infraestrutura e operação.
- Desempenho e escalabilidade.
- Confiabilidade e tratamento de falhas.
- Segurança.
- Limitações conhecidas da solução.

Diferencie claramente consequências discutidas na reunião de riscos ou impactos inferidos a partir das decisões registradas.

Não transforme riscos identificados em requisitos ou decisões já aprovadas.

### 3.8. Decisões relacionadas

Apresente as ADRs que fundamentam a proposta.

Para cada ADR, inclua:
- Identificador.
- Título.
- Link relativo para o arquivo Markdown correspondente.

Utilize os nomes reais dos arquivos disponibilizados. Considere que o RFC ficará em `docs/RFC.md` e construa os links relativos corretamente, assumindo que as ADRs estão no mesmo diretório, salvo indicação contrária.

## 4. Diretrizes de qualidade

O documento final deve atender aos seguintes critérios:

- **Concisão:** aproximadamente 2 a 4 páginas. Evite conteúdo redundante e explicações desnecessárias.
- **Nível arquitetural:** apresente decisões e justificativas, não instruções de implementação.
- **Rastreabilidade:** as afirmações relevantes devem ser sustentadas pelas ADRs correspondentes.
- **Consistência:** não deve haver contradições entre a proposta consolidada e as decisões registradas.
- **Neutralidade:** o documento deve apresentar a proposta para revisão, sem tratar questões ainda abertas como decisões definitivas.
- **Clareza:** deve ser compreensível para integrantes dos times técnico e de produto.

Não tente preencher todas as seções com conteúdo artificial. Se determinada informação não estiver disponível nas ADRs, indique isso explicitamente.

## 5. Critérios obrigatórios de validação

Antes de finalizar o documento, verifique:

- [ ] O RFC apresenta metadados, incluindo status, data, autor e revisores.
- [ ] O resumo executivo explica claramente a proposta.
- [ ] O contexto explica o problema de negócio e a motivação da iniciativa.
- [ ] A proposta técnica consolida as decisões arquiteturais das ADRs.
- [ ] Existem pelo menos duas alternativas reais discutidas e descartadas na reunião, acompanhadas dos respectivos trade-offs.
- [ ] A seção "Questões em aberto" contém pelo menos dois pontos efetivamente adiados ou não decididos na reunião.
- [ ] Os impactos e riscos estão identificados e não são confundidos com decisões.
- [ ] O documento referencia, por meio de links relativos funcionais, pelo menos duas ADRs do pacote.
- [ ] Nenhuma decisão, justificativa ou alternativa foi inventada.
- [ ] O RFC não duplica o detalhamento que pertence ao FDD.
- [ ] O documento possui aproximadamente 2 a 4 páginas.

## 6. Entrega

Crie efetivamente o arquivo Markdown em:

`docs/RFC.md`

Não se limite a apresentar o conteúdo do documento na resposta.

Ao finalizar:
1. Verifique se o arquivo foi criado no caminho solicitado.
2. Revise o conteúdo contra todos os critérios de validação.
3. Disponibilize um link para download do arquivo RFC.md.
4. Apresente uma confirmação sucinta da entrega, indicando eventuais limitações encontradas nas fontes.

---

## FDD

v0

Me ajude a refinar o prompt abaixo para que eu envie para o Claude Sonnet 5.5 e tenha um arquivo de FDD conforme os requisitos abaixo. Minha ideia é copiar ele literalmente e simplesmente enviar em outro modelo em uma conversa do zero dando acesso apenas as ADRs em formato de markdown.

<prompt>

Gere um arquivo markdown de Feature Design Docs (FDD) com as decisões formalizadas e a proposta de acordo com as ADRs, a RFC em anexo e o código do projeto disponível em https://github.com/Manfrin92/mba-ia-desafio-design-docs-com-ia folder "src".

- Seção "Contratos públicos" inclui pelo menos 4 endpoints HTTP com payload de exemplo (request e response) e status codes
- Seção "Integração com o sistema existente" referencia pelo menos 4 caminhos de arquivo reais do código base e descrever como o módulo de webhooks vai se integrar com cada um (por exemplo, como o método `changeStatus` será estendido, como as classes de erro existentes serão reutilizadas)
- Seção "Observabilidade" cita métricas, logs e tracing
- Detalhe o como implementar da feature precisa estar acionável o suficiente para um desenvolvedor pegar e começar a codar. 

Minimo que deve conter,

- Contexto e motivação técnica
- Objetivos técnicos
- Escopo e exclusões
- Fluxos detalhados (criação do evento na outbox, processamento pelo worker, retry, DLQ)
- Contratos públicos (endpoints HTTP com payloads de exemplo, headers, status codes, semântica)
- Matriz de erros previstos com códigos no padrão `WEBHOOK_*`
- Estratégias de resiliência (timeouts, retries, backoff, fallback)
- Observabilidade (métricas, logs, tracing)
- Dependências e compatibilidade
- Critérios de aceite técnicos
- Riscos e mitigação

</prompt>

---

v1

# Geração de Feature Design Document (FDD) — Webhooks de saída para mudança de status de pedidos

## 1. Papel e objetivo

Atue como um Software Architect sênior, especialista em arquitetura de sistemas, Node.js, TypeScript, Express, Prisma ORM e MySQL.

Sua tarefa é produzir um **Feature Design Document (FDD)** completo, tecnicamente consistente e suficientemente detalhado para orientar a implementação de uma nova feature de webhooks de saída para notificações de mudança de status de pedidos.

O documento deve traduzir as decisões arquiteturais formalizadas nas ADRs em uma especificação técnica de implementação, considerando a RFC e a estrutura real do projeto existente.

O resultado deve ser um documento de engenharia, não uma explicação conceitual sobre webhooks ou padrões arquiteturais.

## 2. Fontes de informação

Você terá acesso às seguintes fontes:

**ADRs:**
Todas as ADRs relacionadas à feature, fornecidas como arquivos Markdown.

**RFC:**
Documento de Request for Comments que consolida o contexto, a proposta técnica, as alternativas consideradas, os impactos e as questões em aberto.

**Repositório:**
https://github.com/Manfrin92/mba-ia-desafio-design-docs-com-ia

O código-fonte relevante está no diretório `src`, mas você deve inspecionar também outros arquivos do repositório quando necessário, especialmente:

- `prisma/schema.prisma`
- `package.json`
- Configurações de ambiente.
- Configurações de testes.
- Scripts de inicialização.
- Configurações de infraestrutura e Docker.

## 3. Regras de análise e rastreabilidade

Antes de elaborar o FDD, analise integralmente as fontes disponíveis.

Siga rigorosamente estas regras:

### 3.1. Precedência das fontes

Utilize a seguinte hierarquia:

1. ADRs com status Accepted/Aceito: decisões arquiteturais formalizadas e obrigatórias para o design.
2. RFC: contexto, requisitos, proposta técnica, restrições e questões em aberto.
3. Código-fonte: evidência da implementação existente, convenções e pontos de integração.

O código existente não deve ser utilizado para contradizer uma decisão arquitetural formalizada.

Caso identifique divergências entre as fontes, registre-as explicitamente.

### 3.2. Não inventar decisões

As ADRs podem conter pontos explicitamente não decididos.

Não transforme hipóteses, inferências ou sugestões em decisões formalizadas.

Para cada ponto relevante, identifique sua natureza:

- **Decidido:** formalizado em uma ADR aceita.
- **Existente:** comportamento ou estrutura comprovadamente presente no código.
- **Proposta do FDD:** solução técnica sugerida para preencher uma lacuna de implementação.
- **Pendente:** decisão que precisa de validação dos responsáveis.

Quando uma definição técnica for necessária para tornar a implementação acionável, apresente uma proposta concreta, identifique-a como proposta do FDD e explique sua justificativa.

Não apresente essa proposta como se tivesse sido aprovada na reunião ou formalizada em uma ADR.

### 3.3. Rastreabilidade

Sempre que uma decisão arquitetural influenciar uma definição de implementação, referencie a ADR correspondente.

Utilize referências no formato `[ADR-001]`, `[ADR-002]` etc.

Não atribua decisões a uma ADR que não as formalizou.

### 3.4. Integridade das fontes

Não invente informações sobre o código-fonte. Todo caminho de arquivo, classe, função, método, interface, tipo ou propriedade citado como existente deve ser confirmado no repositório.

Caso não consiga acessar o repositório, informe essa limitação e não fabrique referências de código.

## 4. Diretrizes de elaboração

O FDD deve ser orientado à implementação.

Evite explicações genéricas sobre conceitos como Transactional Outbox, HMAC, polling, retry e DLQ.

Priorize:

- Estruturas de dados concretas.
- Responsabilidades de módulos e componentes.
- Fluxos de execução detalhados.
- Contratos HTTP.
- Assinaturas e responsabilidades de métodos.
- Alterações necessárias no código existente.
- Regras de negócio e validações.
- Tratamento de erros.
- Comportamentos em cenários de falha.
- Estratégias de testes.
- Critérios objetivos de aceite.

Sempre que possível, utilize tabelas para apresentar estruturas, contratos, estados, responsabilidades e matrizes de decisão.

Inclua diagramas Mermaid quando contribuírem para o entendimento dos fluxos e relacionamentos.

Não introduza novas tecnologias, serviços externos ou dependências de infraestrutura sem justificativa e sem verificar sua compatibilidade com as ADRs.

## 5. Estrutura obrigatória do FDD

O documento deve conter, no mínimo, as seguintes seções.

### 5.1. Metadados

- Título da feature.
- Tipo de documento: Feature Design Document.
- Status: Proposto para revisão.
- Referências às ADRs utilizadas.
- Referência à RFC.

### 5.2. Contexto e motivação técnica

Descreva:

- O problema atual que motiva a feature.
- Como o sistema funciona atualmente.
- Por que a solução é necessária.
- Quais limitações técnicas e arquiteturais precisam ser respeitadas.
- Quais requisitos de negócio e não funcionais influenciam a implementação.

### 5.3. Objetivos técnicos

Liste os objetivos técnicos da implementação de forma objetiva e verificável.

### 5.4. Escopo e exclusões

Documente:

- Funcionalidades contempladas.
- Funcionalidades explicitamente fora do escopo.
- Limitações conhecidas.
- Comportamentos que não devem ser implementados nesta entrega.

Não amplie o escopo por iniciativa própria.

### 5.5. Arquitetura e componentes

Descreva os componentes necessários e suas respectivas responsabilidades.

Considere, no mínimo, quando aplicável:

- Módulo de gerenciamento de webhooks.
- Integração com o módulo de pedidos.
- Persistência da Outbox.
- Worker independente.
- Processador de eventos.
- Cliente HTTP de entrega.
- Gerenciamento de secrets e assinatura HMAC.
- Persistência da Dead Letter Queue.
- Mecanismo de replay administrativo.

Explique as interações entre os componentes e as dependências existentes.

Inclua um diagrama Mermaid da arquitetura proposta.

### 5.6. Modelo de dados e persistência

Especifique detalhadamente as estruturas de persistência necessárias.

Para cada tabela, apresente:

- Nome.
- Finalidade.
- Colunas.
- Tipos de dados compatíveis com Prisma e MySQL.
- Obrigatoriedade e valores default.
- Chaves primárias e estrangeiras.
- Índices.
- Relacionamentos.
- Restrições e regras de integridade.

Considere especificamente:

- Configurações de endpoints de webhook.
- Eventos da Outbox.
- Dead Letter Queue.
- Histórico de tentativas de entrega, caso necessário.

Explique a relação entre as entidades e a cardinalidade proposta.

Inclua exemplos de registros quando ajudarem a esclarecer o modelo.

Apresente os estados possíveis da Outbox e da DLQ e suas respectivas transições.

Diferencie claramente as estruturas já existentes das novas estruturas propostas.

Quando houver decisões não formalizadas nas ADRs, identifique as propostas e os pontos pendentes.

### 5.7. Fluxos detalhados de execução

Documente passo a passo os principais fluxos da feature.

No mínimo:

**A. Criação do evento na Outbox**
- Entrada no fluxo de mudança de status.
- Execução da transação.
- Atualização do pedido.
- Registro do histórico de status.
- Aplicação do filtro de status dos webhooks.
- Construção do snapshot do payload.
- Inserção na Outbox.
- Commit ou rollback.

**B. Processamento pelo Worker**
- Inicialização do processo.
- Conexão com o banco.
- Polling.
- Seleção dos eventos elegíveis.
- Ordenação.
- Controle do estado de processamento.
- Identificação dos endpoints destinatários.
- Construção dos headers.
- Assinatura HMAC-SHA256.
- Envio HTTP.
- Interpretação da resposta.
- Atualização do estado.

**C. Falhas e retry**
- Identificação de timeout.
- Identificação de erros de conexão.
- Tratamento de respostas HTTP.
- Atualização do contador de tentativas.
- Cálculo do próximo horário de execução.
- Backoff.
- Retorno do evento à fila de processamento.

**D. Dead Letter Queue**
- Critério para esgotamento das tentativas.
- Transferência do evento.
- Preservação dos dados necessários para debug.
- Consulta de eventos na DLQ.
- Replay manual.
- Autorização ADMIN.
- Auditoria da operação.

**E. Gerenciamento dos webhooks**
- Criação.
- Consulta.
- Atualização.
- Remoção ou desativação.
- Configuração de status de interesse.
- Geração de secret.
- Rotação de secret.

Inclua diagramas Mermaid de sequência para os fluxos mais relevantes.

### 5.8. Contratos públicos

Especifique os contratos HTTP da feature.

Inclua pelo menos quatro endpoints HTTP, com detalhamento suficiente para que frontend, backend e consumidores externos possam implementar suas integrações.

Considere as operações de:

- Gerenciamento de endpoints de webhook.
- Consulta de configurações.
- Atualização ou remoção de configurações.
- Rotação de secret.
- Consulta do histórico de entregas.
- Consulta e replay de eventos na DLQ.

Para cada endpoint, documente:

| Informação | Descrição |
|---|---|
| Método e rota | Verbo HTTP e caminho completo |
| Objetivo | Responsabilidade do endpoint |
| Autenticação | JWT ou outro mecanismo definido |
| Autorização | Roles exigidas |
| Headers | Headers obrigatórios e opcionais |
| Path/query params | Parâmetros e validações |
| Request | Payload JSON completo de exemplo |
| Response | Payload JSON completo de exemplo |
| Status codes | Códigos HTTP e respectivas condições |
| Erros | Códigos de erro e mensagens |
| Semântica | Comportamento esperado e efeitos da operação |

Utilize o prefixo de rotas efetivamente adotado pelo projeto, verificando sua composição no código.

Além dos endpoints de gerenciamento, documente o contrato HTTP enviado ao cliente destinatário do webhook:

- Estrutura do payload.
- Headers enviados.
- Significado de cada header.
- Formato da assinatura HMAC.
- Como o cliente deve validar a assinatura.
- Como identificar e deduplicar eventos.
- Comportamento esperado em caso de duplicidade.
- Semântica de entrega e tratamento de respostas HTTP.

Se alguma definição não estiver formalizada nas fontes, apresente uma proposta explícita e identifique-a como pendente de aprovação.

### 5.9. Matriz de erros

Crie uma matriz de erros previstos para o módulo.

Utilize o padrão de nomenclatura `WEBHOOK_*`.

Para cada erro, documente:

- Código.
- Significado.
- Condição que provoca o erro.
- Camada responsável.
- Comportamento esperado.
- HTTP status, quando aplicável.
- Se deve gerar retry, DLQ ou rejeição imediata.

Considere erros de validação, autenticação, autorização, persistência, configuração, assinatura, comunicação HTTP, timeout, processamento e replay.

Reutilize os padrões e classes de erro existentes no projeto sempre que possível.

Não invente classes existentes. Caso seja necessária uma nova classe, identifique-a como proposta.

### 5.10. Estratégias de resiliência

Detalhe as estratégias de:

- Timeout HTTP.
- Retry.
- Backoff exponencial.
- Limite de tentativas.
- Classificação de falhas transitórias e permanentes.
- Recuperação de eventos em processamento após queda do worker.
- Comportamento em caso de indisponibilidade do banco.
- Shutdown gracioso.
- Prevenção de processamento duplicado.
- Ordenação dos eventos.
- Comportamento durante a indisponibilidade prolongada de um endpoint.

Respeite as políticas formalizadas nas ADRs.

Quando houver lacunas, apresente alternativas ou uma proposta recomendada, sem alterar silenciosamente as decisões aceitas.

### 5.11. Segurança

Documente:

- Autenticação e autorização da API de gerenciamento.
- Geração de secrets.
- Armazenamento e acesso às secrets.
- Assinatura HMAC-SHA256.
- Validação da assinatura pelo destinatário.
- Rotação de secrets.
- Período de convivência entre secrets.
- Proteção de dados sensíveis em logs.
- Validação de URLs.
- Restrições de tamanho do payload.
- Riscos relacionados à configuração de endpoints externos.

Dê atenção especial às lacunas de segurança identificadas nas ADRs, sem apresentar hipóteses como decisões já aprovadas.

### 5.12. Integração com o sistema existente

Esta seção é obrigatória e deve ser especialmente detalhada.

Identifique de 4 a 6 caminhos de arquivos reais do código-fonte e explique como cada um será utilizado ou modificado para implementar a feature.

Para cada arquivo, apresente:

| Informação | Descrição |
|---|---|
| Caminho | Caminho real no repositório |
| Responsabilidade atual | O que o arquivo faz atualmente |
| Integração necessária | Como participará da feature |
| Alterações propostas | O que deverá ser modificado |
| Novos elementos | Métodos, funções, classes ou tipos necessários |
| Dependências | Componentes com os quais interage |

Considere especialmente:

- `OrderService.changeStatus`: como o evento será publicado dentro da transação existente, preservando atomicidade.
- Tratamento de erros: como as classes e middlewares existentes serão reutilizados.
- Autenticação: como os middlewares existentes serão aproveitados.
- Logger: como serão adicionados logs e redaction de informações sensíveis.
- Prisma: como serão adicionados os novos modelos e operações.
- Rotas: como o módulo será registrado na composição atual da aplicação.
- Worker: como será criado o novo entry point e como compartilhará as configurações necessárias.
- Testes: como a infraestrutura existente será estendida.

Não se limite aos exemplos acima.

Utilize os arquivos efetivamente encontrados no repositório.

Para cada alteração, detalhe o ponto de extensão e o comportamento esperado.

Quando possível, identifique os métodos existentes e proponha assinaturas de novos métodos, indicando parâmetros, retornos e responsabilidades.

**Não apresente apenas uma lista de arquivos. O objetivo é fornecer um plano concreto de integração com o código existente.**

### 5.13. Organização proposta dos arquivos

Apresente a estrutura de diretórios e arquivos que deverão existir após a implementação.

Diferencie:

- Arquivos existentes que serão modificados.
- Novos arquivos.
- Arquivos de configuração.
- Arquivos de migrations.
- Arquivos de testes.

Respeite o padrão de organização por módulos e as convenções encontradas no projeto.

Não proponha uma reorganização geral do código que não seja necessária para a feature.

### 5.14. Observabilidade

Especifique as estratégias de observabilidade considerando métricas, logs e tracing.

**Métricas:**
- Quantidade de eventos criados.
- Quantidade de eventos pendentes.
- Quantidade de eventos processados.
- Quantidade de entregas com sucesso.
- Quantidade de falhas.
- Quantidade de eventos na DLQ.
- Quantidade de retries.
- Latência de entrega.
- Tempo de processamento.
- Idade do evento mais antigo pendente.
- Saúde do worker.

Para cada métrica, especifique o que mede e sua utilidade operacional.

**Logs:**
- Eventos criados.
- Início e término de processamento.
- Tentativas de entrega.
- Respostas HTTP.
- Falhas e timeouts.
- Movimentações para DLQ.
- Operações de replay.
- Rotação de secrets.

Identifique os campos estruturados recomendados e os dados que nunca devem ser registrados.

**Tracing:**
Explique como correlacionar a mudança de status, o registro na Outbox, o processamento pelo worker e a entrega HTTP.

Considere a propagação de identificadores de correlação entre processos.

Não presuma que exista uma infraestrutura de tracing instalada. Verifique o projeto e diferencie recursos existentes de propostas.

### 5.15. Dependências e compatibilidade

Documente:

- Dependências npm existentes que serão reutilizadas.
- Novas dependências necessárias, se houver.
- Recursos nativos do Node.js que podem ser aproveitados.
- Compatibilidade com a versão atual do Node.js.
- Compatibilidade com Prisma e MySQL.
- Configurações de ambiente necessárias.
- Scripts de execução.
- Impactos na inicialização e no deploy.

Evite adicionar dependências sem necessidade técnica.

### 5.16. Estratégia de testes

Defina uma estratégia de testes cobrindo:

- Testes unitários.
- Testes de integração.
- Testes de persistência.
- Testes transacionais.
- Testes do worker.
- Testes de retry e backoff.
- Testes de DLQ e replay.
- Testes de assinatura HMAC.
- Testes de contratos HTTP.
- Testes de autorização.
- Testes de falhas e recuperação.

Para cada grupo, apresente cenários concretos e o comportamento esperado.

Considere os padrões de testes já existentes no projeto.

### 5.17. Critérios de aceite técnicos

Crie critérios objetivos e verificáveis.

Cada critério deve permitir determinar se a implementação está correta.

Inclua requisitos de:

- Atomicidade entre mudança de status e evento.
- Snapshot do payload.
- Processamento assíncrono.
- Latência esperada.
- Retry.
- DLQ.
- Replay administrativo.
- Autenticação e assinatura.
- Deduplicação.
- Segurança.
- Observabilidade.
- Compatibilidade com o sistema existente.

### 5.18. Riscos e mitigação

Documente os riscos identificados nas fontes e os riscos técnicos adicionais encontrados durante a análise.

Para cada risco, apresente:

- Descrição.
- Causa.
- Impacto.
- Probabilidade qualitativa, se possível.
- Estratégia de mitigação.
- Se é uma limitação aceita, uma pendência ou um risco que exige aprovação.

Diferencie riscos formalmente identificados nas ADRs de riscos adicionais identificados durante o design.

### 5.19. Decisões pendentes e questões para revisão

Consolide todas as decisões necessárias para iniciar a implementação que ainda não estejam formalizadas.

Para cada item, apresente:

- Questão.
- Contexto.
- Por que precisa ser decidida.
- Proposta recomendada, quando possível.
- Impacto de não decidir.
- ADR relacionada, quando aplicável.

Não esconda ambiguidades para produzir um documento aparentemente completo.

## 6. Requisitos de qualidade

O documento final deverá atender aos seguintes requisitos:

1. Deve ser tecnicamente consistente com todas as ADRs aceitas.
2. Deve respeitar os requisitos e restrições da RFC.
3. Deve refletir a estrutura real do código-fonte.
4. Deve possuir pelo menos quatro endpoints HTTP documentados com exemplos completos de request e response.
5. Deve referenciar pelo menos quatro caminhos reais de arquivos existentes e explicar suas integrações.
6. Deve detalhar os fluxos de Outbox, Worker, Retry e DLQ.
7. Deve especificar os modelos de dados e suas transições de estado.
8. Deve documentar métricas, logs e tracing.
9. Deve fornecer detalhes suficientes para que um desenvolvedor consiga iniciar a implementação sem precisar tomar decisões arquiteturais importantes por conta própria.
10. Deve identificar explicitamente as decisões ainda pendentes.
11. Deve evitar abstrações desnecessárias e propostas que contrariem as ADRs.
12. Deve manter rastreabilidade entre requisitos, decisões e implementação.

## 7. Revisão de consistência antes da entrega

Antes de finalizar o documento, faça uma revisão crítica verificando:

- Se todas as ADRs disponíveis foram consideradas.
- Se todas as decisões aceitas foram respeitadas.
- Se existem contradições entre seções do FDD.
- Se os contratos HTTP são consistentes com os modelos de dados.
- Se os estados da Outbox e DLQ são coerentes com os fluxos descritos.
- Se os exemplos de payload correspondem aos contratos definidos.
- Se os códigos de erro estão consistentes.
- Se os caminhos de arquivos e símbolos citados existem.
- Se as propostas estão claramente diferenciadas das decisões formalizadas.
- Se as questões pendentes foram preservadas.
- Se os critérios de aceite são verificáveis.
- Se um desenvolvedor consegue compreender o que precisa modificar e implementar.

Corrija inconsistências identificadas antes de entregar.

## 8. Entrega

Gere um arquivo Markdown independente, com todo o conteúdo do FDD.

Nome do arquivo:

`FDD-webhooks-de-saida-para-mudanca-de-status-de-pedidos.md`

Não entregue apenas um resumo, um esqueleto ou uma lista de recomendações.

O documento deve conter todas as seções obrigatórias, tabelas, exemplos de payloads, diagramas e especificações técnicas.

Salve o arquivo como artefato para download e disponibilize um link para acessá-lo, caso o ambiente permita a criação de arquivos.

Na resposta final, apresente somente uma breve confirmação da geração do documento e o link para download. Não reproduza o conteúdo integral do FDD na resposta.

---

## PRD

---
v0

Me ajude a refinar o prompt abaixo para que eu envie para o Claude Sonnet 5.5 e tenha um arquivo de PRD conforme os requisitos abaixo. Minha ideia é copiar ele literalmente e simplesmente enviar em outro modelo em uma conversa do zero dando acesso as 6 ADRs na pasta "docs/adrs", RFC.md dentro de "docs", FDD.md dentro de "docs" no repositório https://github.com/Manfrin92/mba-ia-desafio-design-docs-com-ia 

- Como ele é mais alto nível, com RFC, FDD e ADRs em mãos vira praticamente uma consolidação.
- Identifica no mínimo 8 requisitos funcionais discutidos na reunião
- Inclui pelo menos 1 objetivo com métrica e meta quantitativa
- Seção "Fora de escopo" lista pelo menos 2 itens explicitamente descartados ou adiados na reunião
- Seção "Riscos" inclui pelo menos 2 riscos com probabilidade, impacto e mitigação
- Cobre a feature de Sistema de Webhooks de Notificação de Pedidos. 

<sessoes-minimo>
- Resumo e contexto da feature
- Problema e motivação
- Público-alvo e cenários de uso
- Objetivos e métricas de sucesso
- Escopo (incluso e fora de escopo)
- Requisitos funcionais
- Requisitos não funcionais
- Decisões e trade-offs principais
- Dependências
- Riscos e mitigação
- Critérios de aceitação
- Estratégia de testes e validação
</sessoes-minimo>

---

v1

# Criação do PRD — Sistema de Webhooks de Notificação de Pedidos

## Contexto

Você deve produzir o Product Requirements Document (PRD) da feature **Sistema de Webhooks de Notificação de Pedidos**, pertencente a um Order Management System (OMS) existente.

Você terá acesso ao seguinte repositório:

https://github.com/Manfrin92/mba-ia-desafio-design-docs-com-ia

Utilize exclusivamente os seguintes documentos como fontes de informação:

- `docs/adrs/`: as 6 Architecture Decision Records (ADRs) existentes.
- `docs/RFC.md`: Request for Comments da feature.
- `docs/FDD.md`: Feature Design Document da feature.

Não utilize outros arquivos do repositório como fontes de requisitos ou decisões. Não consulte fontes externas.

## Objetivo

Produzir o arquivo `docs/PRD.md`, consolidando as informações presentes nas ADRs, RFC e FDD em um documento de requisitos de produto.

O PRD deve apresentar uma visão de produto e negócio da feature, respondendo principalmente:

- Qual problema estamos resolvendo?
- Por que essa feature é necessária?
- Quem são seus usuários e beneficiários?
- O que a solução precisa oferecer?
- Quais resultados esperamos alcançar?
- Como saberemos se a feature foi bem-sucedida?

O PRD deve ser escrito em nível mais alto de abstração do que o RFC e o FDD. Não deve reproduzir detalhes de implementação, estruturas internas, algoritmos, contratos técnicos ou decisões de infraestrutura que pertencem aos documentos técnicos.

## Diretrizes de elaboração

### 1. Consolidação das fontes

Leia integralmente as 6 ADRs, o RFC e o FDD antes de começar a escrever.

Utilize as fontes de acordo com suas responsabilidades:

- **ADRs:** decisões arquiteturais consolidadas e seus principais trade-offs.
- **RFC:** contexto, motivação, proposta geral, alternativas e restrições.
- **FDD:** funcionalidades, comportamentos esperados, fluxos, regras de negócio, requisitos não funcionais e critérios técnicos.

Evite duplicar conteúdo entre os documentos.

Quando uma informação aparecer em mais de uma fonte, consolide-a em uma única descrição coerente.

### 2. Rastreabilidade e confiabilidade

Todo requisito, objetivo, restrição, risco ou decisão apresentado no PRD deve possuir fundamento identificável em pelo menos uma das fontes fornecidas.

Não invente requisitos, regras de negócio, métricas, decisões ou restrições.

Quando houver divergências entre as fontes:

- Priorize decisões explicitamente formalizadas nas ADRs com status Accepted.
- Considere o RFC como referência para a proposta geral.
- Utilize o FDD para detalhar comportamentos e requisitos funcionais.
- Não resolva conflitos por suposição.
- Registre divergências relevantes como pontos de atenção.

Não afirme que algo foi discutido ou decidido na reunião se essa informação não estiver representada nos documentos disponíveis.

Quando não houver informação suficiente para preencher alguma seção, declare explicitamente a lacuna.

### 3. Nível de abstração

O PRD deve ser compreensível para Product Managers, stakeholders de negócio, arquitetos e desenvolvedores.

Descreva o comportamento e o valor esperado da feature, não sua implementação interna.

Por exemplo:

- Prefira: "O sistema deve permitir que clientes externos recebam notificações quando o status de um pedido for alterado."
- Evite: "O worker deve consultar periodicamente a tabela de outbox no MySQL utilizando polling."

O segundo exemplo pertence ao FDD.

As decisões arquiteturais devem ser resumidas apenas quando forem relevantes para explicar restrições, capacidades ou trade-offs da solução.

## Estrutura obrigatória do PRD

O documento deve conter todas as seguintes seções:

### 1. Resumo e contexto da feature

Apresente uma visão executiva da feature, explicando sua finalidade, contexto no OMS e principais capacidades oferecidas.

### 2. Problema e motivação

Descreva o problema que motivou o desenvolvimento da feature, suas limitações atuais e o valor esperado com sua implementação.

### 3. Público-alvo e cenários de uso

Identifique os principais usuários, consumidores ou sistemas beneficiados pela feature.

Apresente cenários de uso concretos, explicando quem utiliza a funcionalidade, em qual situação e qual resultado espera obter.

### 4. Objetivos e métricas de sucesso

Defina os objetivos da feature e como seu sucesso será medido.

Obrigatoriamente, inclua pelo menos um objetivo acompanhado de uma métrica e uma meta quantitativa.

Sempre que possível, utilize indicadores e metas presentes nas fontes.

Caso não exista uma meta quantitativa explicitamente documentada, proponha uma meta razoável, identificando-a claramente como **meta sugerida a validar**, e não como uma decisão já aprovada.

As métricas devem ser relevantes para o resultado da feature, mensuráveis e verificáveis.

### 5. Escopo

Divida esta seção em:

**5.1. Dentro do escopo**

Liste as capacidades e funcionalidades contempladas nesta entrega.

**5.2. Fora de escopo**

Liste pelo menos 2 itens explicitamente descartados ou adiados nas discussões documentadas nas fontes.

Diferencie claramente itens descartados de itens adiados para fases futuras.

Não classifique funcionalidades como fora de escopo apenas porque não aparecem no FDD ou RFC. Deve existir evidência de exclusão ou adiamento.

### 6. Requisitos funcionais

Identifique pelo menos 8 requisitos funcionais distintos, fundamentados nas fontes.

Cada requisito deve:

- Possuir identificador único, no formato `PRD-FR-XX`.
- Apresentar descrição clara e objetiva.
- Descrever um comportamento ou capacidade que o produto deve oferecer.
- Ser verificável, evitando expressões vagas como "deve ser eficiente" ou "deve funcionar corretamente".

Organize os requisitos em grupos funcionais quando isso melhorar a compreensão.

Não divida artificialmente uma única funcionalidade em vários requisitos apenas para atingir a quantidade mínima.

### 7. Requisitos não funcionais

Apresente os requisitos relacionados às características de qualidade da solução, como:

- Confiabilidade.
- Disponibilidade.
- Desempenho.
- Segurança.
- Escalabilidade.
- Observabilidade.

Inclua somente características que tenham fundamento nas fontes disponíveis.

Sempre que possível, descreva requisitos de forma mensurável. Não invente metas técnicas.

### 8. Decisões e trade-offs principais

Resuma as principais decisões arquiteturais registradas nas ADRs e seus impactos do ponto de vista do produto.

Explique brevemente os benefícios e as limitações relevantes de cada decisão.

Inclua referências aos respectivos arquivos ADR.

Não reproduza o conteúdo integral das ADRs.

### 9. Dependências

Identifique dependências funcionais, técnicas ou externas necessárias para que a feature alcance seus objetivos.

Diferencie dependências já existentes de dependências que precisam ser disponibilizadas ou implementadas.

### 10. Riscos e mitigação

Identifique pelo menos 2 riscos relevantes para o sucesso da feature.

Para cada risco, apresente:

- Descrição.
- Probabilidade: Baixa, Média ou Alta.
- Impacto: Baixo, Médio ou Alto.
- Estratégia de mitigação.

Priorize riscos que possam afetar a experiência dos consumidores dos webhooks, a confiabilidade das notificações, a segurança ou a operação da solução.

Os riscos devem estar fundamentados nas fontes ou ser identificados explicitamente como riscos derivados de suas decisões e restrições.

### 11. Critérios de aceitação

Defina critérios objetivos que permitam verificar se a feature atende aos requisitos de produto.

Os critérios devem representar resultados observáveis, preferencialmente organizados por funcionalidade ou requisito funcional.

Evite reproduzir testes unitários, detalhes de implementação ou cenários excessivamente técnicos do FDD.

### 12. Estratégia de testes e validação

Descreva, em nível de produto, como validar que a feature atende às necessidades identificadas.

Considere, quando aplicável:

- Validação dos principais cenários de uso.
- Validação dos requisitos funcionais.
- Validação dos requisitos não funcionais.
- Testes de integração com consumidores externos.
- Validação das métricas de sucesso.
- Critérios para considerar a feature pronta para disponibilização.

Não reproduza o plano técnico detalhado de testes do FDD.

## Requisitos de qualidade

O documento final deve atender aos seguintes critérios:

1. Ser escrito integralmente em português brasileiro.
2. Utilizar Markdown.
3. Ter linguagem profissional, objetiva e consistente.
4. Ser compreensível para stakeholders técnicos e não técnicos.
5. Conter todas as 12 seções obrigatórias.
6. Identificar pelo menos 8 requisitos funcionais distintos.
7. Apresentar pelo menos um objetivo com métrica e meta quantitativa.
8. Apresentar pelo menos 2 itens explicitamente descartados ou adiados na seção Fora de escopo.
9. Apresentar pelo menos 2 riscos com probabilidade, impacto e mitigação.
10. Manter consistência com as decisões registradas nas ADRs, RFC e FDD.
11. Evitar redundância com os documentos técnicos.
12. Não apresentar suposições como fatos ou decisões aprovadas.

## Revisão final

Antes de finalizar, faça uma revisão crítica do PRD:

- Todas as funcionalidades estão alinhadas ao RFC e ao FDD?
- As decisões arquiteturais estão consistentes com as ADRs?
- Existem pelo menos 8 requisitos funcionais realmente distintos?
- Existe pelo menos um objetivo com métrica e meta quantitativa?
- Os itens fora de escopo possuem evidência explícita de descarte ou adiamento?
- Existem pelo menos 2 riscos com probabilidade, impacto e mitigação?
- Os critérios de aceitação permitem verificar os resultados esperados?
- O documento está no nível correto de abstração para um PRD?
- Existem informações inventadas, contraditórias ou desnecessariamente técnicas?

Corrija os problemas identificados antes de entregar.

## Entregável

Produza exclusivamente o arquivo:

`docs/PRD.md`

Não altere, crie ou sobrescreva nenhum outro arquivo do repositório.

Após concluir:

1. Salve o documento no caminho especificado.
2. Informe brevemente que a geração foi concluída.
3. Disponibilize um link para download do arquivo `PRD.md`.
4. Apresente eventuais lacunas de informação ou critérios que não puderam ser atendidos com evidências nas fontes fornecidas.

Não apresente apenas o conteúdo do PRD na conversa. O arquivo Markdown deve ser efetivamente criado e disponibilizado para download.

---

## Tracker

---

v0

Me ajude a refinar o prompt abaixo para que eu envie para o Claude Sonnet 5.5 e tenha um arquivo de Tracker conforme os requisitos abaixo. Minha ideia é copiar ele literalmente e simplesmente enviar em outro modelo em uma conversa do zero dando acesso as 6 ADRs na pasta "docs/adrs", RFC.md dentro de "docs", FDD.md dentro de "docs", PRD dentro de "docs", transcricao na raiz do projejeto TRANSCRICAO.md no repositório, codigo fonte dentro de "src" https://github.com/Manfrin92/mba-ia-desafio-design-docs-com-ia 

- Pelo menos 80% dos itens identificáveis dos documentos têm linha correspondente
- Pelo menos 70% das linhas têm Fonte = `TRANSCRICAO` com timestamp válido no formato `[hh:mm] Nome`
- Pelo menos 5 linhas têm Fonte = `CODIGO` com caminho de arquivo real

Produza o arquivo `docs/TRACKER.md`, uma tabela markdown que mapeia cada item registrado nos seus documentos à origem na transcrição ou no código. O tracker funciona como uma referência cruzada: permite que qualquer leitor entenda de onde veio cada decisão, requisito ou restrição, e garante que a documentação está alinhada com o que foi efetivamente discutido e com o que existe no código.

Formato obrigatório da tabela:

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |

Onde:

- **ID**: identificador único do item (ex: PRD-FR-01, RFC-ALT-02, FDD-CONTRATO-03, ADR-002)
- **Documento**: arquivo onde o item aparece (`docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md`, `docs/adrs/ADR-002-...md`)
- **Tipo**: Requisito Funcional, Requisito Não Funcional, Decisão, Restrição, Trade-off, entre outros
- **Conteúdo (resumo)**: descrição de uma linha do item
- **Fonte**: `TRANSCRICAO` ou `CODIGO`
- **Localização**: para `TRANSCRICAO`, timestamp + nome do falante (ex: `[09:17] Diego`). Para `CODIGO`, caminho do arquivo (ex: `src/modules/orders/order.service.ts`).

---

v1

# Criação do TRACKER — Sistema de Webhooks de Notificação de Pedidos

## Contexto

Você deve produzir o documento **TRACKER.md** para o projeto de implementação do Sistema de Webhooks de Notificação de Pedidos, pertencente a um Order Management System (OMS) existente.

Você terá acesso ao seguinte repositório:

https://github.com/Manfrin92/mba-ia-desafio-design-docs-com-ia

Utilize exclusivamente os seguintes artefatos como fontes de informação:

- `TRANSCRICAO.md` — transcrição das discussões que deram origem às decisões e aos requisitos.
- `docs/adrs/` — as 6 Architecture Decision Records (ADRs).
- `docs/RFC.md` — Request for Comments.
- `docs/FDD.md` — Functional Design Document.
- `docs/PRD.md` — Product Requirements Document.
- `src/` — código-fonte da aplicação.

Não utilize fontes externas, conhecimento presumido ou informações que não estejam presentes nesses artefatos.

## Objetivo

Produzir o arquivo `docs/TRACKER.md`, contendo uma tabela de rastreabilidade que mapeie os itens identificáveis presentes nos documentos de arquitetura, requisitos e design à sua respectiva origem na transcrição ou no código-fonte.

O Tracker deve funcionar como uma referência cruzada entre a documentação e suas evidências, permitindo que qualquer leitor:

1. Identifique onde cada requisito, decisão, restrição ou trade-off está documentado.
2. Localize a discussão original que motivou determinado item, quando houver evidência na transcrição.
3. Identifique a implementação correspondente no código-fonte, quando aplicável.
4. Verifique se a documentação está fundamentada no que foi efetivamente discutido e implementado.

## 1. Análise dos artefatos

Antes de produzir o arquivo, examine integralmente os artefatos disponibilizados.

Identifique os itens rastreáveis em:

- PRD: requisitos funcionais, requisitos não funcionais, regras de negócio, restrições e critérios de aceitação.
- RFC: alternativas avaliadas, decisões propostas, justificativas, trade-offs e restrições arquiteturais.
- FDD: funcionalidades, fluxos, contratos, comportamentos, integrações e regras técnicas.
- ADRs: decisões arquiteturais, contexto, alternativas consideradas, consequências e restrições.

Considere também que um mesmo assunto pode aparecer em mais de um documento. Cada ocorrência documental deve possuir sua própria linha no Tracker, com o identificador e o documento correspondentes.

Não limite a análise aos exemplos acima. Inclua todos os itens relevantes que possam ser rastreados.

## 2. Rastreamento das fontes

Para cada item identificado, procure sua evidência na transcrição e no código-fonte.

### Fonte: TRANSCRICAO

Utilize `TRANSCRICAO.md` como fonte primária para identificar a origem dos requisitos, decisões, justificativas e restrições discutidos durante as conversas.

A localização deve conter obrigatoriamente:

- Timestamp original da transcrição.
- Nome do falante correspondente.

Formato obrigatório:

`[hh:mm] Nome`

Exemplo:

`[09:17] Diego`

Regras:

- Utilize exclusivamente timestamps e nomes de falantes efetivamente presentes na transcrição.
- Não invente, estime ou arredonde timestamps.
- Não atribua uma fala a alguém diferente de seu autor original.
- Se um item for discutido em diferentes momentos, escolha a ocorrência mais representativa de sua origem.
- Quando possível, prefira a fala que apresenta a motivação ou justificativa original, em vez de uma simples confirmação posterior.

### Fonte: CODIGO

Utilize o diretório `src/` para identificar evidências de funcionalidades, comportamentos, contratos e decisões efetivamente presentes na implementação.

A localização deve conter o caminho relativo real do arquivo dentro do repositório.

Exemplo:

`src/modules/orders/order.service.ts`

Regras:

- Verifique a existência de cada arquivo antes de referenciá-lo.
- Não invente caminhos, nomes de arquivos, classes, funções ou módulos.
- Utilize arquivos que efetivamente contenham evidências relacionadas ao item rastreado.
- Não associe um requisito à implementação apenas porque o nome de um arquivo parece relacionado.
- Quando possível, indique o arquivo mais diretamente relacionado ao comportamento implementado.

Importante: a existência de uma implementação não comprova, por si só, que ela foi originalmente discutida. Da mesma forma, uma discussão na transcrição não comprova que a funcionalidade foi implementada.

Preserve essa distinção ao selecionar a fonte de cada linha.

## 3. Requisitos quantitativos de rastreabilidade

O Tracker deve atender aos seguintes critérios:

**Cobertura documental:**
- Pelo menos 80% dos itens identificáveis nos documentos analisados devem possuir uma linha correspondente no Tracker.
- Considere como denominador todos os itens rastreáveis identificados no PRD, RFC, FDD e nas 6 ADRs.
- Cada item deve ser contabilizado individualmente, inclusive quando houver itens semelhantes em documentos diferentes.
- Priorize a cobertura de todos os itens relevantes, evitando omissões.

**Rastreabilidade da transcrição:**
- Pelo menos 70% das linhas do Tracker devem possuir `Fonte = TRANSCRICAO`.
- Todas essas linhas devem conter timestamp e nome de falante válidos, exatamente conforme aparecem na transcrição.
- Não utilize timestamps fictícios ou estimados para atingir esse percentual.

**Rastreabilidade do código:**
- Pelo menos 5 linhas devem possuir `Fonte = CODIGO`.
- Todas devem apontar para arquivos existentes em `src/`.
- As associações devem representar evidências reais da implementação.

Os percentuais devem ser calculados com base no número total de linhas de dados da tabela, desconsiderando o cabeçalho.

Se algum critério quantitativo não puder ser atingido com as evidências disponíveis, não fabrique informações para satisfazê-lo. Registre a limitação de forma transparente.

## 4. Formato obrigatório do documento

O arquivo deve conter uma tabela Markdown com exatamente as seguintes colunas e nesta ordem:

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Resumo do requisito | TRANSCRICAO | [09:17] Diego |
| RFC-ALT-01 | docs/RFC.md | Trade-off | Resumo da alternativa avaliada | TRANSCRICAO | [10:32] Diego |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | Resumo do contrato técnico | CODIGO | src/modules/orders/order.service.ts |
| ADR-002 | docs/adrs/ADR-002-exemplo.md | Decisão | Resumo da decisão arquitetural | TRANSCRICAO | [11:45] Diego |

Os exemplos acima são ilustrativos. Não os utilize como dados reais.

### Definição das colunas

**ID**

Identificador único do item rastreado.

Sempre que possível, preserve o identificador original utilizado no documento de origem.

Exemplos:
- PRD-FR-01
- PRD-RNF-01
- RFC-ALT-02
- FDD-CONTRATO-03
- ADR-002

Caso o item não possua identificador explícito, crie um identificador consistente e único, utilizando o prefixo do documento e uma numeração sequencial.

**Documento**

Caminho relativo do documento em que o item foi identificado.

Exemplos:
- `docs/PRD.md`
- `docs/RFC.md`
- `docs/FDD.md`
- `docs/adrs/ADR-002-titulo.md`

**Tipo**

Classificação do item rastreado.

Utilize classificações adequadas ao conteúdo, como:
- Requisito Funcional
- Requisito Não Funcional
- Regra de Negócio
- Decisão
- Restrição
- Trade-off
- Alternativa
- Contrato
- Fluxo
- Critério de Aceitação
- Consequência Arquitetural

**Conteúdo (resumo)**

Descrição objetiva, em uma única linha, do requisito, decisão ou item rastreado.

Não copie parágrafos inteiros dos documentos. Preserve o significado original.

**Fonte**

Aceita exclusivamente os seguintes valores:

- `TRANSCRICAO`
- `CODIGO`

**Localização**

Para `TRANSCRICAO`, utilize o timestamp e o nome do falante.

Para `CODIGO`, utilize o caminho relativo real do arquivo dentro de `src/`.

## 5. Regras de qualidade

Observe rigorosamente as seguintes regras:

1. Não invente evidências para preencher lacunas de rastreabilidade.
2. Não crie itens que não estejam presentes nos documentos analisados.
3. Não utilize a documentação como fonte de evidência. O campo Fonte deve apontar para a transcrição ou para o código-fonte.
4. Não utilize conhecimento externo para inferir decisões ou requisitos.
5. Não associe uma decisão a uma fala apenas porque os assuntos são semelhantes. Deve existir uma relação semântica clara.
6. Evite referências genéricas como `[09:17] Diego` quando a fala não sustenta o item rastreado.
7. Não utilize caminhos de arquivos presumidos ou inexistentes.
8. Não elimine itens relevantes apenas para facilitar o atendimento aos percentuais.
9. Não agrupe diferentes requisitos ou decisões em uma única linha quando puderem ser rastreados individualmente.
10. Evite linhas duplicadas para o mesmo item dentro do mesmo documento.
11. Preserve a correspondência entre os identificadores e os documentos originais.
12. Não considere uma funcionalidade implementada quando o código apenas contém estruturas preparatórias ou comentários sem implementação efetiva.

## 6. Validação final

Antes de finalizar o arquivo, realize uma verificação de qualidade.

Verifique:

- Se todos os documentos previstos foram analisados, incluindo as 6 ADRs.
- Se pelo menos 80% dos itens identificáveis possuem correspondência no Tracker.
- Se pelo menos 70% das linhas possuem `Fonte = TRANSCRICAO`.
- Se todas as localizações de transcrição possuem timestamps e falantes válidos.
- Se existem pelo menos 5 linhas com `Fonte = CODIGO`.
- Se todos os caminhos de código referenciados realmente existem.
- Se os identificadores são únicos e consistentes.
- Se não existem linhas sem evidência verificável.
- Se a tabela está formatada corretamente em Markdown.

Calcule os percentuais efetivamente alcançados. Não apresente estimativas.

## 7. Entrega

Crie o arquivo:

`docs/TRACKER.md`

O documento deve conter:

1. Um título `# Tracker`.
2. Uma breve descrição do objetivo do documento.
3. A tabela de rastreabilidade no formato obrigatório.
4. Uma seção final `## Validação de Rastreabilidade`, apresentando:
   - Quantidade total de itens identificados nos documentos.
   - Quantidade de itens rastreados.
   - Percentual de cobertura documental.
   - Quantidade total de linhas no Tracker.
   - Quantidade e percentual de linhas com fonte `TRANSCRICAO`.
   - Quantidade de linhas com fonte `CODIGO`.
   - Resultado da validação dos caminhos de código.
   - Indicação de atendimento ou não de cada requisito quantitativo.

Não inclua explicações extensas ou conteúdo que não contribua para a rastreabilidade.

Salve o arquivo diretamente em `docs/TRACKER.md`, sem modificar os demais documentos ou o código-fonte.

Ao concluir, disponibilize um link para download do arquivo gerado e apresente um resumo objetivo dos resultados da validação.