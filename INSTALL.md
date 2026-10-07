# Instalação da skill `plan-task`

Instruções para o **agente** (Claude Code ou outro agente compatível com skills no formato `SKILL.md`) instalar esta skill na máquina do usuário. Siga na ordem.

## Conteúdo do pacote

```
INSTALL.md                  este arquivo (não instalar)
plan-task/
  SKILL.md                  definição da skill (frontmatter name/description + instruções)
  templates/
    task-doc.md             template do doc principal da task
    spec.md                 template de cada spec
```

Só a pasta `plan-task/` é instalada. O nome da pasta precisa continuar `plan-task` (igual ao `name:` do `SKILL.md`).

## 1. Escolher o escopo

Pergunte ao usuário se não estiver claro:

| Escopo | Destino | Quando |
|---|---|---|
| **Usuário** (padrão) | `<HOME>/.claude/skills/plan-task/` | Disponível em todos os projetos |
| **Projeto** | `<raiz-do-repo>/.claude/skills/plan-task/` | Só naquele repo; pode ser versionado e compartilhado com o time |

`<HOME>` por sistema:

| Sistema | `<HOME>` | Destino (escopo usuário) |
|---|---|---|
| Windows | `%USERPROFILE%` (ex.: `C:\Users\fulano`) | `%USERPROFILE%\.claude\skills\plan-task\` |
| Linux (Arch, Ubuntu, Debian, Fedora…) | `$HOME` | `~/.claude/skills/plan-task/` |
| macOS | `$HOME` | `~/.claude/skills/plan-task/` |
| WSL | `$HOME` **do Linux**, não o do Windows | `~/.claude/skills/plan-task/` |

Se `CLAUDE_CONFIG_DIR` estiver definida, use `$CLAUDE_CONFIG_DIR/skills/plan-task/` no lugar de `<HOME>/.claude/skills/plan-task/`.

## 2. Verificar instalação existente

Antes de copiar, confira se o destino já existe. **Se existir, não sobrescreva sem confirmação do usuário** — ele pode ter uma versão editada. Ofereça backup (`plan-task.bak-<data>`) ou cancelar.

## 3. Instalar

Assuma que o zip foi extraído (ou será) num diretório temporário `<PACOTE>` que contém `INSTALL.md` e `plan-task/`.

### Windows (PowerShell)

```powershell
$zip  = "C:\caminho\para\plan-task.zip"
$tmp  = Join-Path $env:TEMP "plan-task-pkg"
$dest = Join-Path $env:USERPROFILE ".claude\skills"

Expand-Archive -Path $zip -DestinationPath $tmp -Force
New-Item -ItemType Directory -Force -Path $dest | Out-Null
if (Test-Path "$dest\plan-task") { throw "Ja existe $dest\plan-task — confirme com o usuario antes de sobrescrever" }
Copy-Item -Recurse "$tmp\plan-task" "$dest\plan-task"
Remove-Item -Recurse -Force $tmp
```

Git Bash no Windows: use o bloco Linux abaixo (`$HOME` aponta para `C:\Users\<usuario>`).

### Linux / macOS / WSL

```bash
ZIP="/caminho/para/plan-task.zip"
TMP="$(mktemp -d)"
DEST="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/skills"

unzip -q "$ZIP" -d "$TMP"
mkdir -p "$DEST"
if [ -e "$DEST/plan-task" ]; then
  echo "Ja existe $DEST/plan-task — confirme com o usuario antes de sobrescrever"
else
  cp -r "$TMP/plan-task" "$DEST/plan-task"
fi
rm -rf "$TMP"
```

Se `unzip` não existir:

| Distro | Comando |
|---|---|
| Arch / Manjaro | `sudo pacman -S unzip` |
| Ubuntu / Debian / Mint | `sudo apt install unzip` |
| Fedora / RHEL | `sudo dnf install unzip` |
| openSUSE | `sudo zypper install unzip` |
| Alpine | `sudo apk add unzip` |
| macOS | já vem instalado |

Alternativas sem instalar nada: `python3 -m zipfile -e "$ZIP" "$TMP"` ou `bsdtar -xf "$ZIP" -C "$TMP"`.

Peça confirmação do usuário antes de rodar `sudo`.

### Escopo projeto

Mesmo processo, trocando o destino por `<raiz-do-repo>/.claude/skills`. Pergunte ao usuário se a pasta deve ser commitada.

## 4. Validar

1. Estrutura no destino:
   - `plan-task/SKILL.md`
   - `plan-task/templates/task-doc.md`
   - `plan-task/templates/spec.md`
2. Primeira linha do `SKILL.md` é `---` e o frontmatter tem `name: plan-task`.
3. Arquivos em UTF-8 (há acentos e emojis de status: ⬜ 🔄 ✅ ⏸️ ⏭️ ❌). Não converta encoding nem finais de linha.
4. Linux/macOS: permissões de leitura para o usuário (`chmod -R u+rX "$DEST/plan-task"` se necessário).
5. Diga ao usuário para **reiniciar a sessão** do Claude Code (skills são carregadas na inicialização). Depois, `/plan-task` deve aparecer na lista de skills; teste com `/plan-task TESTE-1 --no-commit` num repo qualquer.

## 5. Desinstalar

Remover a pasta `plan-task` do destino escolhido e reiniciar a sessão.

## Observações sobre a skill

- Cria planos em `docs/tasks/<ID>/` no repo onde for usada. Recomende ao usuário adicionar `docs/tasks/` ao `.gitignore` do projeto — a skill trata a pasta como artefato local e nunca a commita.
- Usa `git` (detecta o ID pela branch e faz commits em lote). Sem `git` instalado, funciona só com ID passado como argumento e `--no-commit`.
- Flags: `--no-commit` (sem commits automáticos), `--no-approve` (pula aprovação do plano).
