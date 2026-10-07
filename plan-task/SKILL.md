---
name: plan-task
description: Fragmenta uma solicitação de desenvolvimento em subtarefas rastreáveis, documenta o progresso em docs/tasks/[nome_da_task].md e faz commit atômico por subtarefa validada em execução real. Use ao receber qualquer pedido de implementação, refatoração ou correção que envolva mais de um passo, ou quando o usuário pedir para planejar, quebrar em tarefas, rastrear progresso ou retomar uma task já planejada.
argument-hint: [descrição da solicitação de desenvolvimento]
---

# plan-task

Recebe uma solicitação de desenvolvimento, quebra em subtarefas granulares e mantém um
documento vivo do progresso. O documento é a fonte da verdade: é atualizado durante a execução,
não no final.

## Invocação sem descrição

Se a skill for chamada sem descrição da solicitação, procure em `docs/tasks/` uma task já
iniciada e retome-a a partir do `## ⏸ CHECKPOINT`. Havendo mais de uma candidata, liste as
opções e pergunte qual retomar.

## Esqueleto do arquivo

O documento tem sempre esta ordem, desde a criação:

```
cabeçalho → subtarefas (em ordem de ID) → ## O que deu errado → ## ⏸ CHECKPOINT
```

**Ao criar um documento novo, copie `template.md` (na pasta desta skill) e preencha** — não
reconstrua o formato de memória. Em caso de dúvida sobre nível de detalhe, consulte
`exemplo.md`, na mesma pasta, que traz um documento preenchido com os cinco status em uso.

Sem contadores de progresso no cabeçalho: a lista de subtarefas já é a fonte da verdade, e um
resumo numérico mantido à mão dessincroniza do estado real.

## 1. Localizar o documento

Caminho obrigatório, relativo à raiz do projeto (onde está `.git`, ou o diretório de trabalho
se não houver repositório):

```
docs/tasks/[nome_da_task].md
```

`[nome_da_task]` em kebab-case, derivado da solicitação (ex.: `importacao-csv-clientes`).

**Antes de criar qualquer arquivo, liste o conteúdo de `docs/tasks/` com as ferramentas de
leitura.** Se algum arquivo existente tiver correspondência lógica com a solicitação atual —
ainda que o nome não seja idêntico ao que você geraria — é **obrigatório** retomar esse arquivo
em vez de criar outro. `importacao-csv.md` e `importacao-de-csv-clientes.md` são a mesma task;
duas entradas para a mesma demanda é falha de rastreamento, não organização.

Havendo mais de um candidato plausível, pergunte ao usuário qual retomar antes de escrever.
Ao retomar, leia o `## ⏸ CHECKPOINT` no fim do arquivo e continue a partir dele.

## 2. Fragmentar e alinhar

Quebre a solicitação em subtarefas lógicas e granulares. Cada subtarefa deve ser:

- **Independentemente verificável** — dá para dizer "isso funciona" sem depender de outra em aberto.
- **Pequena** — se a validação exigir mais de um cenário de teste distinto, provavelmente são duas subtarefas.
- **Commitável sozinha** — se não dá para commitar isolada sem quebrar o build, está grande demais.
- **Ordenada por dependência** — declare as dependências explicitamente no campo próprio.

IDs sequenciais e estáveis: `T1`, `T2`, `T3`… IDs nunca são reaproveitados nem renumerados;
subtarefas descobertas depois entram no fim (`T7`), mesmo que pertençam logicamente ao meio.

**Alinhe antes de executar.** Escreva o documento com todas as subtarefas em `Pendente`,
apresente a lista ao usuário e espere o aceite antes da primeira linha de implementação.
Plano errado aprovado cedo custa uma frase; plano errado descoberto tarde custa uma branch
inteira de commits. Só pule o aceite se o usuário já tiver mandado executar direto.

## 3. Estrutura obrigatória de cada subtarefa

Reproduza este bloco exatamente, um por subtarefa:

```markdown
## [ID] Título da subtarefa
- **Status:** Pendente | Em andamento | Bloqueado | Cancelado | Concluído
- **Descrição:**
- **Critério de aceite:**
- **Validação executada:** (o que foi executado, o que foi observado, data)
- **Commit:** (hash curto + título, ou —)
- **Dependências:**
- **Notas:**
```

Regras de preenchimento:

- **Status** — um único valor, sem adornos.
  - `Bloqueado` — algo externo impede o avanço e **alguém precisa agir**. Explique o quê.
  - `Cancelado` — o escopo mudou e a subtarefa não será feita. Não apague o bloco: registre o
    motivo em **Notas**, preservando o histórico da decisão.
- **Critério de aceite** — observável e testável. "Retorna os dados corretos" não serve;
  "`GET /clientes/{id}` de um id existente retorna `200` com `nome`, `email` e `criado_em` preenchidos" serve.
- **Validação executada** — vazio até existir execução real. Nunca preencha com o teste que
  você *pretende* rodar.
- **Notas** — curto, no máximo 2 ou 3 linhas, para contexto de decisão. Erros e tentativas
  frustradas **não** vão aqui: vão para `## O que deu errado` (seção 7).

### Tamanho da evidência

**Validação executada** é um resumo, não um dump. Registre o que foi executado, o resultado
observado e as **chaves ou sinais principais verificados** — omita arrays longos, dados
repetitivos e campos irrelevantes ao critério de aceite:

```
POST /clientes → 201; corpo com id (uuid), nome, email, criado_em (ISO-8601). 2026-08-14
```

Se o payload ou a saída forem complexos e valer a pena preservá-los, grave em
`docs/tasks/evidencias/[ID].json` (ou `.txt`) e referencie:

```
GET /relatorios?periodo=2026-07 → 200; 412 itens, chaves validadas: total, itens[].valor,
itens[].data. Evidência: docs/tasks/evidencias/T4.json. 2026-08-14
```

Redija segredos antes de gravar evidência: tokens, senhas, chaves de API e dados pessoais reais
saem do arquivo (substitua por `***`).

## 4. Regra de transição de status (estrita)

Uma subtarefa só vira **Concluído** depois de **observar o comportamento real em execução**.
Não basta o código existir: é preciso rodar e olhar o resultado.

Nunca conclui subtarefa:

- Código escrito e "parece certo".
- Compilação, type-check ou lint passando.
- Teste unitário com a dependência mockada.
- Resultado previsto ou simulado em vez de resultado observado.

### Forma da validação por tipo de subtarefa

| Tipo | O que conta como validação |
|---|---|
| Endpoint HTTP | requisição real ao serviço **em execução** + inspeção do corpo da resposta |
| CLI / script | execução real + inspeção do stdout e do exit code |
| Migração de banco | rodar a migração + consultar o schema/dados resultantes |
| UI | renderizar e interagir de fato com a tela |
| Job / worker / fila | disparar e inspecionar o efeito produzido |
| Biblioteca / módulo | exercitar a função com entradas reais e conferir a saída |

Para endpoints HTTP, vale a regra específica: **status `200` isolado não conclui nada.** Um
`200` com corpo vazio, `data: null`, erro embutido no payload ou campos faltando é reprovação.
O equivalente em outros tipos: exit code `0` sem saída esperada, migração que roda sem criar a
coluna, tela que monta sem os dados — tudo reprovação.

### Ambiente de validação

Valide contra ambiente **local ou de desenvolvimento**. Nunca contra produção. Se a validação
criar dados, limpe-os ao final ou use dados claramente marcados como de teste. Se o único
ambiente disponível for produção, pare e pergunte ao usuário.

Se não houver ambiente executável (sem serviço subível, sem credenciais, sem base), a subtarefa
vai para `Bloqueado` com o motivo e o usuário é avisado. Nunca promova por conveniência.

### Quando a validação falha

Falha é qualquer resultado fora do critério de aceite — erro, timeout, ou sucesso aparente com
resultado incorreto. Nesse caso:

1. **Não conclua a subtarefa.** Mantenha o status em `Em andamento`.
2. Registre uma linha em `## O que deu errado` com o ID, o esperado e o obtido. Sem colar payload.
3. Corrija e valide de novo.

Se três ciclos de correção→validação passarem **sem que a natureza do erro mude**, pare de
tentar: mude para `Bloqueado`, registre o impedimento e leve a questão ao usuário. Erro que
muda a cada tentativa é progresso; erro idêntico três vezes é loop, e insistir em silêncio
queima contexto e esconde o problema.

## 5. Commit atômico por subtarefa

Cada subtarefa concluída rende **um commit, e apenas um**. Regras:

- Commite **somente** quando o status virar `Concluído` — ou seja, depois da validação real.
  Código não validado não entra no histórico como se estivesse pronto.
- O commit inclui a mudança de código, a evidência em `docs/tasks/evidencias/` (se houver) **e**
  a atualização do `docs/tasks/[nome_da_task].md` — tudo no mesmo commit.
- Nada de outras subtarefas junto. Se o `git status` mostrar arquivos modificados que não
  pertencem a esta subtarefa, adicione ao índice (`git add`) apenas os arquivos estritamente
  relacionados a ela.
- Se o repositório estiver na branch padrão (`main`/`master`), **crie uma branch antes** do
  primeiro commit da task.
- **Nunca faça push** por iniciativa própria. Push só quando o usuário pedir explicitamente.
- Registre o hash curto no campo **Commit** da subtarefa.

### Template obrigatório de mensagem de commit

Padrão Conventional Commits. Título imperativo de 5 a 6 palavras, linha em branco, e um
parágrafo curto (1 a 3 linhas) explicando a mudança técnica ou o motivo da implementação.

```text
<tipo>: <resumo imperativo curto>

<parágrafo curto com detalhes ou o motivo da implementação>
```

Exemplo:

```text
feat: valida CPF no cadastro

Implementa a chamada à API externa para validar o formato e a situação cadastral do CPF.
Evidência salva e tipagem ajustada no payload.
```

O ID da subtarefa **não** entra na mensagem — a rastreabilidade fica no campo **Commit** do
documento, que guarda o hash. Se o repositório já tiver convenção própria de mensagem, ela
prevalece sobre este template.

Se o projeto não for um repositório git, pule esta seção e anote `—` no campo **Commit**.

## 6. Checkpoint e persistência

O checkpoint é o que permite retomar a task numa sessão futura. Grave-o:

- **Ao concluir cada subtarefa** — este é o gatilho principal, e coincide com o commit atômico.
  Assim o documento nunca fica muito à frente do último ponto seguro.
- **Ao pausar por qualquer motivo** — fim de sessão, bloqueio, pergunta ao usuário.
- **Ao perceber o contexto se aproximando do limite** (margem de 10%), pausando antes de
  continuar a executar. Este gatilho é secundário e não confiável: a janela pode ser compactada
  sem aviso, então nunca dependa só dele para persistir estado.

Ao gravar: confira que status e notas das subtarefas refletem o estado real — se a regra de
atualizar a cada mudança de status foi seguida, isso é só uma conferência, não uma reescrita.
Depois substitua o bloco de checkpoint anterior (vale sempre o mais recente):

```markdown
## ⏸ CHECKPOINT — [AAAA-MM-DD HH:MM]
- **Parou em:** [ID e o ponto exato dentro da subtarefa]
- **Feito nesta sessão:** [resumo objetivo]
- **Próximo passo:** [primeira ação concreta da retomada]
- **Contexto necessário para retomar:** [arquivos, comandos, decisões tomadas, valores/IDs em uso]
- **Estado do git:** [branch, último commit da task, se há trabalho não commitado]
- **Pendências abertas:** [dúvidas ou bloqueios que precisam de resposta do usuário]
```

Ao pausar de fato, avise o usuário que a execução parou e onde o estado ficou salvo.

## 7. Seção "O que deu errado"

Seção única e cronológica com os tropeços do desenvolvimento. Concentra o ruído que de outro
modo inflaria as **Notas** de cada subtarefa.

```markdown
## O que deu errado
- **[ID]** [o que se esperava] → [o que aconteceu]. Causa: [raiz, se identificada]. Resolvido em: [como, ou "em aberto"]
```

Regras:

- **Uma linha por ocorrência.** Sem stack trace completo, sem payload colado, sem log bruto.
  Se o detalhe importar, referencie `docs/tasks/evidencias/`.
- Registre o que custou tempo ou mudou o rumo: validação reprovada, premissa errada sobre a API,
  dependência que não se comportou como esperado, retrabalho.
- Não registre digitação corrigida na hora nem erro de sintaxe trivial.
- Nunca apague linhas antigas — o valor da seção é evitar repetir o mesmo erro numa sessão futura.

## Durante a execução

- Atualize o documento **a cada mudança de status**, não em lote no final.
- Descobriu trabalho novo? Crie a subtarefa no documento antes de executá-la.
- Uma subtarefa nunca some do documento: vira `Cancelado` com o motivo, jamais é apagada.
