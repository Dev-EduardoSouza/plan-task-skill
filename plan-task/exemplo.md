<!--
Exemplo preenchido de docs/tasks/[nome_da_task].md, para calibrar nível de detalhe.
Não é um arquivo de trabalho: serve de referência de formato.
-->

# Importação de clientes via CSV

**Solicitação original:** Permitir que o admin suba um CSV de clientes e importe em lote, com relatório de linhas rejeitadas.
**Criada em:** 2026-08-10
**Última atualização:** 2026-08-14

## T1 Criar tabela de importações e migração
- **Status:** Concluído
- **Descrição:** Tabela `importacoes` guardando arquivo, status do processamento e contadores de linhas.
- **Critério de aceite:** Migração aplica sem erro e a tabela existe com as colunas `id`, `arquivo`, `status`, `linhas_ok`, `linhas_erro`, `criado_em`.
- **Validação executada:** `alembic upgrade head` → ok; `\d importacoes` no psql confirma as 6 colunas e o índice em `status`. 2026-08-11
- **Commit:** `a3f91c2` — feat: cria tabela de importações
- **Dependências:** —
- **Notas:** —

## T2 Endpoint de upload do arquivo
- **Status:** Concluído
- **Descrição:** `POST /importacoes` recebe multipart, persiste o arquivo e cria o registro em status `pendente`.
- **Critério de aceite:** Upload de CSV válido retorna `201` com `id` e `status: "pendente"`; arquivo fica acessível no storage.
- **Validação executada:** `POST /importacoes` com clientes.csv (30 linhas) → 201; corpo com id (uuid), status "pendente", criado_em (ISO-8601). Arquivo confirmado no bucket local. 2026-08-12
- **Commit:** `7d20b8e` — feat: recebe upload de csv de clientes
- **Dependências:** T1
- **Notas:** Limite de 10 MB definido no nginx, não na aplicação.

## T3 Processar o CSV e gravar clientes
- **Status:** Em andamento
- **Descrição:** Worker lê o arquivo pendente, valida linha a linha e insere os clientes válidos.
- **Critério de aceite:** Após processar um CSV com 30 linhas (28 válidas, 2 inválidas), a importação fica `concluida` com `linhas_ok: 28` e `linhas_erro: 2`, e os 28 clientes existem no banco.
- **Validação executada:** —
- **Commit:** —
- **Dependências:** T2
- **Notas:** Inserção em lotes de 500 para não estourar a transação.

## T4 Relatório de linhas rejeitadas
- **Status:** Bloqueado
- **Descrição:** `GET /importacoes/{id}/rejeitadas` devolve as linhas recusadas com o motivo.
- **Critério de aceite:** Retorna `200` com lista de objetos contendo `linha`, `conteudo` e `motivo`, uma entrada por linha rejeitada.
- **Validação executada:** —
- **Commit:** —
- **Dependências:** T3
- **Notas:** Bloqueada até T3 gravar os motivos de rejeição — não há o que consultar antes disso.

## T5 Notificar o admin por e-mail ao concluir
- **Status:** Cancelado
- **Descrição:** Disparar e-mail ao admin quando a importação terminar.
- **Critério de aceite:** —
- **Validação executada:** —
- **Commit:** —
- **Dependências:** T3
- **Notas:** Cancelada em 2026-08-13 — o usuário decidiu que a tela de acompanhamento já resolve, e não há serviço de e-mail configurado no projeto.

## O que deu errado
- **T2** Esperava `201` no upload → veio `413 Request Entity Too Large` com arquivo de 12 MB. Causa: limite do nginx, não da aplicação. Resolvido em: `client_max_body_size` ajustado para 20m e documentado no README.
- **T3** Esperava 28 clientes inseridos → inseriu 28 mas com `criado_em` nulo. Causa: o default estava só no modelo ORM, não na coluna, e a inserção em lote passa por baixo do ORM. Resolvido em: default movido para a migração.
- **T3** Esperava lote de 500 linhas processar em uma transação → estourou timeout do pool de conexões com CSV de 5.000 linhas. Causa: em aberto, suspeita de conexão não devolvida ao pool entre lotes.

## ⏸ CHECKPOINT — 2026-08-14 17:40
- **Parou em:** T3, investigando o timeout do pool com arquivos grandes. CSV pequeno (30 linhas) já passa; o de 5.000 falha.
- **Feito nesta sessão:** T3 corrigida quanto ao `criado_em` nulo e revalidada com o arquivo de 30 linhas. Reproduzido o timeout com o arquivo grande.
- **Próximo passo:** Instrumentar `importacoes/worker.py:84` para logar conexões ativas do pool entre lotes e confirmar se são devolvidas.
- **Contexto necessário para retomar:** Fixtures em `tests/fixtures/clientes_5000.csv`. Worker sobe com `make worker`. API em `localhost:8000`, banco de dev em `localhost:5432` (docker compose). Importação de teste em curso: id `4c1e...9a`.
- **Estado do git:** branch `feat/importacao-csv`, último commit da task `7d20b8e` (T2). T3 com alterações não commitadas em `worker.py` e na migração.
- **Pendências abertas:** Confirmar com o usuário se linha inválida deve abortar o lote inteiro ou só ser pulada — hoje está pulando.
