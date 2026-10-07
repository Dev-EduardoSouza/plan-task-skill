# <ID> — <título curto>

**Status:** ⬜ Pendente · **Branch:** `<branch>` · **Atualizado:** <AAAA-MM-DD>

## Objetivo

<2-3 frases: o que existe no fim e por quê. Para quem não viu a conversa.>

## O que será feito

<1-2 frases com as peças que serão criadas/alteradas. Ex.: "CRUD de TabelaExemplo: migration, model, repository, service, controller e DTOs.">

```
<contrato resumido, uma linha por item — ex.: POST /tabela-exemplo/{banco} → cria (payload na spec 02)>
```

## Decisões

- <decisão> — <motivo em meia linha>

## Dúvidas abertas

- <pergunta> — (a) <opção> · (b) <opção> — proposta: (a)

## Specs

| # | Spec | Status | Commit |
|---|---|---|---|
| 01 | [<título imperativo>](specs/01-<slug>.md) | ⬜ Pendente | |
| 02 | [<título imperativo>](specs/02-<slug>.md) | ⬜ Pendente | |

**Fora do escopo:** <o que não será feito, ou omitir a linha>

**Próximo passo:** <uma ação>

---

# Execução

Seção de trabalho do agente. O usuário não precisa ler para aprovar o plano.

## Contexto técnico

<achados da investigação com `caminho/arquivo:linha`, armadilhas, versões>

## Ambiente

| O quê | Comando | Para |
|---|---|---|
| <serviço> | `<comando exato>` | <specs> |

**Credencial:** <do agents.md/CLAUDE.md, ou "n/a">
**Estado montado na mão:** <seed, env var, container — ou "nenhum">

## Regras

- **Commit:** `<formato do repo, ex.: feat(<id>): <verbo + até 6 palavras>>` + <corpo, ex.: parágrafo único com o porquê, sem bullets>. Lote: ≥ 3 arquivos E ≥ 500 linhas; flush na última spec e antes de pausar.
- **Nunca no commit:** `Co-Authored-By`, menção a IA, `--no-verify`, a pasta `docs/tasks/`, <testes novos, se o repo os ignora>.
- **Código:** sem comentários. <demais regras obrigatórias do agents.md/CLAUDE.md: camadas, testes, migration, idioma>

## Ponto de parada

**Spec atual:** <nn — o que já está pronto nela>
**Próximo passo:** <ação concreta>
**Lote pendente de commit:** <arquivos ou "nenhum">
**Ambiente a subir:** <comandos>
**Contexto que não está no código:** <decisões, armadilhas>

## Notas

<só mudança estrutural do plano: spec adicionada/cancelada/reordenada e o motivo>
