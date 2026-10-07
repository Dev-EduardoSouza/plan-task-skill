---
name: plan-task
description: Cria uma pasta de plano em docs/tasks/<ID>/ com um doc principal e uma spec por etapa, com critérios de aceite, para o usuário validar exatamente o que será feito antes da execução. Nunca presume regra de negócio ou arquitetura — dúvida vai para o usuário. Executa uma spec por vez, commita em lote (mín. 3 arquivos E 500 linhas; nunca os docs nem testes) e grava ponto de parada quando o contexto fica baixo (~10%). Use quando o usuário pedir "planejar task", "criar plano", "/plan-task", retomar uma task de docs/tasks, ou quando a solicitação for grande o bastante para precisar de plano rastreável entre sessões.
---

# plan-task

Cada task vira uma pasta:

```
docs/tasks/<ID>/
  <ID>.md               doc principal — visão geral, decisões, dúvidas, índice de specs, ponto de parada
  specs/
    01-<slug>.md        uma por etapa — o que será feito + execução dela
    02-<slug>.md
```

Cada arquivo serve a dois leitores, em duas camadas:

- **Usuário (topo):** precisa saber exatamente o que vai ser feito *antes* de aprovar. Texto longo esconde o que importa e faz a validação falhar — escreva como documentação boa: frase curta, concreta, sem processo.
- **Agente (abaixo de `# Execução`):** precisa de tudo para executar e retomar sem histórico — comandos, payloads, tentativas falhas, ponto de parada. Aqui detalhe é bem-vindo.

Se a informação muda a decisão do usuário, vai no topo, curta. Se só ajuda a executar ou verificar, vai em `# Execução`. Spec em arquivo próprio mantém o doc principal curto e deixa o agente carregar só a spec em que está trabalhando.

## 0. Flags

- `--no-commit` — nenhum commit automático.
- `--no-approve` — pula a aprovação do plano.

O resto do argumento é o ID/título. Ex.: `/plan-task 2356 --no-commit`.

## 1. Identificar a task

1. Argumento, se houver.
2. Senão, rode `git rev-parse --abbrev-ref HEAD` e extraia o ID (`feat/solicitacao-2356` → `SOLICITACAO-2356`).
3. Sem repo ou branch sem ID (`main`, `develop`): pergunte.

Pasta: `docs/tasks/<ID-MAIUSCULO>/`.

## 2. Plano existente

- **Pasta com specs pendentes** → retomada: leia `## Ponto de parada` e `## Regras` do doc principal, abra a spec atual, releia os arquivos citados, suba o `Ambiente a subir` e diga ao usuário em uma linha o que vai continuar. Vá ao passo 4.
- **Pasta completa** → pergunte se estende o plano ou abre task nova.
- **Arquivo único `docs/tasks/<ID>.md` (formato antigo)** → continue nele no formato em que está; converter para pasta só se o usuário pedir.
- **Nada** → passo 3.

Nunca sobrescreva plano existente sem confirmação.

## 3. Planejar

### Investigar antes de escrever

Leia `agents.md`/`CLAUDE.md` do repo (URL, porta, credencial, comando de subir serviço, convenção de teste — não invente). Investigue o código até poder citar arquivos reais. Dúvida de regra de negócio, arquitetura ou convenção vira pergunta ao usuário, não suposição.

### O teste do topo

Lendo só os topos (doc principal + specs), o usuário consegue dizer "esse campo não", "esse endpoint não", "faltou isso"? Se precisar adivinhar o que vai sair, falta conteúdo. Se precisar atravessar comando e log, sobra ruído.

### Doc principal (`templates/task-doc.md`)

- **Objetivo:** 2-3 frases — o porquê.
- **O que será feito (geral):** 1-2 frases nomeando as peças ("CRUD de X: migration, model, repository, service, controller e DTOs") e, se ajudar, um bloco **resumido** com o contrato principal (uma linha por rota/tabela, apontando a spec). O detalhe completo mora na spec.
- **Decisões:** uma linha cada, com o motivo em meia linha. Detalhe técnico que sustenta a decisão vai em `## Contexto técnico`.
- **Dúvidas abertas:** uma linha cada — pergunta, opções `(a) · (b)`, proposta. Enquanto houver dúvida, o plano não está pronto para aprovar.
- **Specs:** tabela-índice com link, status e commit.
- **Fora do escopo** quando houver tentação óbvia de fazer mais.
- `# Execução`: `## Contexto técnico`, `## Ambiente` (comandos exatos), `## Regras`, `## Ponto de parada`, `## Notas` — o que vale para a task inteira.
- **`## Regras`** é preenchida sempre, a partir do `agents.md`/`CLAUDE.md` (repo e global) e das regras fixas desta skill: formato de commit do repo com exemplo, proibições de commit (`Co-Authored-By`, menção a IA, docs, testes ignorados), código sem comentários e as demais regras obrigatórias do repo. A compactação de contexto pode levar a skill e o `agents.md` embora no meio da execução; o doc é o que sobra, então as regras precisam estar nele.

### Spec (`templates/spec.md`)

Arquivo `specs/<nn>-<slug>.md`, `nn` com dois dígitos na ordem de execução, `slug` curto em kebab-case.

- **Título:** imperativo e específico, no limite de palavras do formato de commit do repo (ex.: até 6) — vira assunto de commit.
- **O que será feito:** o que vai existir no fim, com nome real: migration `V31__CriarTabelaExemplo`, `TabelaExemploService.salvar`, aba "Entregadores", `POST /...`. Toda estrutura nova ou alterada aparece por inteiro em bloco de código: campos de tabela (nome, tipo, nulidade), payload de request/response, valores de enum, colunas de tela/relatório. Proibido o vago: "ajusta o service", "cria os DTOs necessários", "adiciona os campos".
- Cada estrutura aparece **uma vez**, na spec que a cria. Spec seguinte que usa a mesma estrutura referencia ("consome o payload da spec 02") em vez de copiar — repetição dobra o texto sem dar nada novo pra validar, e cópias divergem.
- **Arquivos:** uma linha com os caminhos.
- **Pronto quando:** comportamento observável em linguagem de produto, uma linha.
- Fora do topo: comando, número de linha, versão de ferramenta, saída de teste.
- `# Execução` → **Critérios**, cada um terminando em `— Validar: \`<comando>\`` (ou `— Validar: manual` + o que olhar). É o que o passo 4 vai rodar; sem comando a verificação vira opinião.

Critério bom descreve o resultado observado, não o status:
- Ruim: `Endpoint responde 200`
- Bom: `GET /api/rotas traz cada item com id_rota, descricao e porcentagemDeAcrescimo não nulos — Validar: \`curl -s -i localhost:8888/api/rotas\``

Lista: tamanho e conteúdo. Caminho de erro relevante: critério próprio com status e corpo. Spec que toca API inclui chamada real ao serviço local. Lógica nova inclui teste unitário; atravessou camadas, teste de integração.

Sem limite de quantidade: o número de specs sai do tamanho da task. O que define o corte é a granularidade — cada spec é uma entrega coerente, verificável sozinha, que cabe numa sessão. Spec que mistura entregas sem relação (ex.: migration + tela) se divide; spec que não dá pra verificar sem a seguinte se junta a ela. Ordene por dependência.

Se os templates não estiverem legíveis, siga a mesma estrutura descrita aqui.

### Status

- Task: `⬜ Pendente` | `🔄 Em andamento` | `⏸️ Pausado` | `✅ Concluído`
- Spec: `⬜ Pendente` | `🔄 Em andamento` | `✅ Feito` | `⏭️ Dispensado` | `❌ Cancelado`

Só esses. Sinônimos (`Concluída`, `Concluída no escopo ajustado`) quebram a leitura entre sessões; o detalhe vai em Notas. Status da spec fica no cabeçalho dela **e** na tabela do doc principal — atualize os dois juntos.

### Aprovação

Mostre no chat, nesta ordem: caminho da pasta, **O que será feito** (geral), o **O que será feito** de cada spec (título + blocos de estrutura) e **Dúvidas abertas**. Objetivo, Decisões, Arquivos e `# Execução` ficam só nos arquivos. Aguarde aprovação (salvo `--no-approve`).

## 4. Executar

Uma spec por vez:

1. Marque `🔄 Em andamento` (spec e tabela do doc principal). `git status --short`: alteração alheia pré-existente fica intocada e vai em Notas.
2. Implemente **o que foi aprovado**. Se a execução exigir algo fora do "O que será feito" aprovado — campo, tabela, payload, endpoint ou comportamento diferente — pare e leve ao usuário antes. Ele aprovou aquela estrutura; ajuste silencioso invalida a validação dele. Ajuste trivial (import, arquivo auxiliar óbvio) só atualiza `Arquivos:`.
3. Verifique cada critério rodando o `Validar:`. Contra o ambiente real (suba o `## Ambiente`), lendo o corpo da resposta campo a campo. 200 com corpo vazio, campo nulo ou lista incompleta = não atendido. Só passa com mock = continua `🔄`.
4. Atualize a `# Execução` da spec: checkboxes, **Feito** (solução final, curta), **Observado** (payload/saída real resumida — "passou" não serve a quem retomar), **Não funcionou** (`abordagem → sintoma`, append-only: é o que impede a próxima sessão de repetir o erro).
5. Atualize status na spec e na tabela do doc principal, e o `Próximo passo`.
6. Lote de commit (pulado com `--no-commit`) — ver abaixo.
7. Sobrescreva `## Ponto de parada` do doc principal apontando para a próxima spec.

Ao fim da última spec, diga ao usuário em 2-4 linhas o que foi entregue e o que ficou de fora; não repita os docs.

### Lote de commit

Commits agrupam specs para não poluir o histórico com micro-commits.

- Conte arquivos de código **tocados por specs concluídas desta task e ainda não commitados**. Nunca conte nem commite: a pasta `docs/tasks/<ID>/` (artefato local, não versionado) e testes novos (ficam fora do Git por regra do repo). Sujeira alheia também fica fora.
- Some linhas: `git diff --shortstat -- <arquivos>` para rastreados; `wc -l` para novos.
- Commite quando **≥ 3 arquivos E ≥ 500 linhas**. Só uma das duas não basta.
- Flush mesmo abaixo do limite: ao fechar a última spec e antes de pausar por contexto — trabalho verificado não fica sem commit no fim da sessão.
- Formato: o do `## Regras` (que vem do `agents.md`/`CLAUDE.md`); ele vence qualquer exemplo daqui. Sem convenção no repo, use:

  ```
  feat(2356): Adiciona paginação e filtro de categoria

  Lista de produtos travava com catálogos grandes; a paginação e o filtro por categoria reduzem a carga e a busca.
  ```

  Assunto `feat(<id>): ` + verbo + até 6 palavras. Lote de uma spec: assunto = título dela. Várias: assunto resume a entrega. Corpo opcional, parágrafo único com o porquê, sem lista de arquivos nem bullets.

- Sem `Co-Authored-By` nem menção a IA — o usuário assina os commits. Sem `--no-verify`; hook falhou, corrija a causa.
- Preencha **Commit** de cada spec do lote (arquivo e tabela) com o mesmo hash.

### Abordagem falhou

Não empilhe tentativa sobre código quebrado:

1. `git status --short` e `git diff --stat`.
2. Reverta **arquivo a arquivo**, só os da spec que falhou (`git restore --staged --worktree -- <arquivo>`). Restore em massa apagaria specs concluídas que esperam o lote.
3. Já commitada num lote: reverter histórico só com confirmação do usuário.
4. `git checkout .`, `git reset --hard`, `git clean` só com pedido explícito.
5. Registre em **Não funcionou** na hora, antes de tentar de novo.

Duas abordagens falhas na mesma spec: pare e peça direção.

Se a realidade contradiz o plano (spec desnecessária, etapa faltando), crie/marque a spec, atualize a tabela, registre o motivo em Notas e avise o usuário se mudou algo que ele aprovou. Spec nova entre duas existentes não renumera as outras: use o próximo número livre e reordene só a tabela.

## 5. Parar por contexto baixo

Alvo: ~10% de contexto restante. Sinais: aviso do harness, transcript com muitos resultados grandes, você não lembra as primeiras specs sem reler.

1. Termine a edição em andamento — nada de código pela metade.
2. Atualize a spec atual (Feito parcial, Não funcionou). Task → `⏸️ Pausado`.
3. Flush do lote pendente (salvo `--no-commit`).
4. Sobrescreva `## Ponto de parada` para um leitor sem nenhum histórico: spec atual e o que está pronto nela, próximo passo como ação, lote pendente (deve estar vazio), ambiente a subir com comandos exatos (inclusive estado montado na mão: seed, container, env var), contexto que não está no código.
5. Uma linha ao usuário: parou por contexto, plano em `docs/tasks/<ID>/`, retomar com `/plan-task <ID>`.

## Regras dos docs

- Idioma do usuário. Sem comentários no código gerado.
- Atualize com Edit, nunca reescreva um arquivo inteiro.
- `## Ponto de parada` é sobrescrito (nunca acumula histórico — o histórico está nas specs) e sempre existe; com a task completa, diz "Concluída".
- Seções novas fora do template não entram no topo; se precisar, vão em `# Execução`.
