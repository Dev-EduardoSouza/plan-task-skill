# plan-task

Skill do Claude Code que transforma um pedido de desenvolvimento em plano com specs por etapa (`docs/tasks/<ID>/`), executa uma spec por vez e commita em lote.

## Instalação nesta máquina

`~/.claude/skills/plan-task` é um link simbólico para a pasta `plan-task/` deste repositório, então editar aqui já muda a skill instalada (reinicie a sessão do Claude Code para recarregar):

```bash
ln -s ~/Projetos/dev/plan-task-skill/plan-task ~/.claude/skills/plan-task
```

Em outra máquina: clone o repo e crie o mesmo link, ou siga o [INSTALL.md](INSTALL.md).
