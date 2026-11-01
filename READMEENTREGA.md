
### 6. README com o processo

Estrutura obrigatória do novo README:

- **Workflow adotado**: como você organizou o trabalho. Em que ordem produziu os documentos, como organizou a interação com a IA
- **Iterações e ajustes**: descreva os principais momentos em que a IA gerou algo errado ou superficial e você teve que corrigir. Quantas iterações principais até chegar ao resultado final
- **Como navegar a entrega**: caminho dos arquivos entregues e ordem sugerida de leitura

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
