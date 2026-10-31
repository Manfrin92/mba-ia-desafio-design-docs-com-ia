
### 6. README com o processo

Estrutura obrigatória do novo README:

- **Workflow adotado**: como você organizou o trabalho. Em que ordem produziu os documentos, como organizou a interação com a IA
- **Iterações e ajustes**: descreva os principais momentos em que a IA gerou algo errado ou superficial e você teve que corrigir. Quantas iterações principais até chegar ao resultado final
- **Como navegar a entrega**: caminho dos arquivos entregues e ordem sugerida de leitura

# Sobre o desafio

O desafio era gerar documentação para que desenvolvedores ou agentes de IA implementassem uma ou várias features novas necessárias para atender novos requisitos de clientes. A ideia foi que a documentação em si também seria gerada com IA e isso seria feito a partir de uma transcrição de uma reunião com as pessoas que discutiram a nova feature do ponto de vista de cliente e técnico. Então, com a transcrição bem definida era possível pegar o que foi decidido e comparando com o código fonte existente criar toda documentação que poderia ser usada pelo time que de fato faria a entrega daquela nova funcionalidade. Na vida real eu vejo que essa transcrição ainda pode ser um pouco complicada de se obter, tenho usado o Zoom para resumos similares, mas é bem claro que muita coisa fica super confusa e ele não parece entender bem o que foi dito, da minha experiência prática isso acontece mais em reuniões em inglês que são os idiomas que geralmente trabalho e gravamos esse tipo de resumo. No futuro acho que isso pode evoluir e aí sim termos algo fidedigno que possa ser utilizado da mesma maneira que nesse trabalho. Talvez o que temos hoje possa ser utilizado como ponto de partida para um posterior refinamento antes do envio final ao time que faz a entrega de fato. De qualquer maneira, a possibilidade de conseguir juntar o contexto inteiro do código com reuniões em que se discutem novas funcionalidades em poucos segundos é impressionante, até então sempre foi preciso ter pessoas com esse tipo de contexto em mente participando das reuniões para dar opiniões, mas agora com a IA é possível que isso mude um pouco talvez o próprio agente participando da reunião trazendo os pontos do código que são iguais ou diferente do que se está falando. Outro ponto que me chamou a atenção foi que talvez isso não seja assim tão viável em projetos grandes, projetos em que trabalhei sim seria possível pois praticamente todos são projetos pequenos e que atendem poucos usuários, mas penso que para empresas maiores isso pode acabar sendo um impeditivo - ou seria preciso delimitar a IA para módulos ou partes do código para ter respostas corretas.

# Ferramentas de IA utilizadas

Chat GPT 6, Claude Sonnet 5.5 tudo direto no chat sem agente modificando nada localmente na minha máquina. Usei o primeiro para refinar os prompts que eventualmente mandaria para o segundo, O GPT me ajudou a melhorar o prompt em si e ir entendendo aos poucos o que faltou, tentei fazer da melhor maneira que pensei sem consultas para treinar meu entendimento do que é um bom prompt logo no começo, mas poderia ter usado ele direto desde o início, mas acredito que isso não me ajudaria a treinar nada.

# Workflow adotado

Comecei pelas ADRs. A organização foi conforme explicado na sessão **Ferramentas de IA utilizadas**.

# Prompts customizados
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

v3

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
v4

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