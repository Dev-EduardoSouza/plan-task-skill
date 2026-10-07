# plan-task

Plugin do Claude Code com a skill `plan-task`: transforma um pedido de desenvolvimento em plano com specs por etapa (`docs/tasks/<ID>/`), executa uma spec por vez e commita em lote.

## Instalar

Dentro do Claude Code:

```
/plugin marketplace add Dev-EduardoSouza/plan-task-skill
/plugin install plan-task@plan-task-skill
```

Reinicie a sessão. A skill aparece como `/plan-task:plan-task`.

## Atualizar

```
/plugin marketplace update plan-task-skill
```

Para ativar a atualização automática: `/plugin` → aba **Marketplaces** → `plan-task-skill` → **Enable auto-update**.

## Publicar uma versão nova

1. Edite `skills/plan-task/`.
2. Suba o `version` em `.claude-plugin/plugin.json` (ex.: `2.0.0` → `2.1.0`). O Claude Code só baixa a versão nova quando esse número muda.
3. Commite e faça push.

## Estrutura

```
.claude-plugin/
  plugin.json         manifesto do plugin (nome, versão)
  marketplace.json    faz este repo funcionar como marketplace de um plugin só
skills/plan-task/
  SKILL.md
  templates/
```

Recomendado nos projetos onde a skill for usada: adicionar `docs/tasks/` ao `.gitignore`.
