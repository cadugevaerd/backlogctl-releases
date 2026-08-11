# backlogctl — releases públicas

`backlogctl` é uma CLI local-first para múltiplos backlogs independentes em SQLite. Este repositório público distribui somente binários verificáveis; o código-fonte permanece no repositório privado canônico.

## Release atual

- CLI: **2.4.0**
- DB schema: **5**
- envelope JSON: contract **2**
- import document: contract **3**
- commit usado no build: `5ffe1dfabe26b2756f1df5388a5fda4b9daae5b7`, que é o próprio `main` privado
- [Release v2.4.0](https://github.com/cadugevaerd/backlogctl-releases/releases/tag/v2.4.0)

**Mudança de comportamento nesta versão:** `doctor` é read-only. Ele criava o arquivo de banco ausente e aplicava migrations; agora um caminho inexistente **falha**, nomeando o caminho. Se você dependia do comportamento antigo, use `backlogctl store init --db PATH` antes de diagnosticar, ou `backlogctl update migrate` para migrar um store existente — que é a operação com backup e confirmação. Detalhes em [PROVENANCE.md](PROVENANCE.md).

Cada release contém seis builds, [`SHA256SUMS`](SHA256SUMS) e [`backlogctl-release.json`](backlogctl-release.json). As cópias no `main` correspondem aos assets da release atual, e essa correspondência é verificada baixando os assets publicados e comparando hash a hash.

## Capacidades

```text
Backlogs     create/list/show/edit/archive/bind
Itens        add/list/show/edit/transition/move/reconcile-status/archive
Prioridade   criticidade, posição, contextos expiráveis e decisões auditáveis
Formato      proposta versionada, preview e apply confirmado
Merge        snapshots/revisions, mappings e apply auditável
Import       JSON v3 preview-first, SHA-256 e aplicação atômica
TODO/FIXME   scanner software-only, opt-in e idempotente
Export       JSON, Markdown e visão consolidada
Update       check/install/migrate com SHA, rollback e backup
```

JSON v1 continua sendo migração agent-led separada; não é aceito diretamente pelo import nativo v3.

## Instalação recomendada

### Claude Code

```bash
claude plugin marketplace add cadugevaerd/claude-skills
claude plugin install backlog@claude-skills
```

O hook `SessionStart` instala/reutiliza o asset pinado e informa `BACKLOGCTL_EXECUTABLE=<path>`.

### Codex

```bash
git clone https://github.com/cadugevaerd/codex-skills.git
cd codex-skills
node plugins/backlog/scripts/ensure-backlogctl.js --install-dir "$HOME/.local/bin"
```

Os plugins verificam URL imutável, plataforma, SHA-256 e `backlogctl version` antes da troca atômica.

## Instalação manual — Linux ARM64

```bash
version=v2.4.0
base="https://github.com/cadugevaerd/backlogctl-releases/releases/download/${version}"
curl -fL -O "$base/backlogctl_linux_arm64"
curl -fL -O "$base/SHA256SUMS"
grep '  backlogctl_linux_arm64$' SHA256SUMS | sha256sum -c -
install -m 0755 backlogctl_linux_arm64 "$HOME/.local/bin/backlogctl"
"$HOME/.local/bin/backlogctl" version
```

Assets disponíveis:

| Plataforma | Arquivo |
|---|---|
| Linux x86_64 | `backlogctl_linux_amd64` |
| Linux ARM64 | `backlogctl_linux_arm64` |
| macOS Intel | `backlogctl_darwin_amd64` |
| macOS Apple Silicon | `backlogctl_darwin_arm64` |
| Windows x86_64 | `backlogctl_windows_amd64.exe` |
| Windows ARM64 | `backlogctl_windows_arm64.exe` |

## Primeiros comandos

```bash
BACKLOGCTL="$HOME/.local/bin/backlogctl"
DB="$HOME/.backlog/backlog.db"

"$BACKLOGCTL" --json store init --db "$DB"
"$BACKLOGCTL" --json doctor --db "$DB"
"$BACKLOGCTL" --json backlog create --code APP --name "Aplicação" --profile software --db "$DB"
"$BACKLOGCTL" --json item add --code APP --title "Corrigir webhook" --description "Adicionar retry e validar entrega." --criticality high --category bug --db "$DB"
"$BACKLOGCTL" --json item list --code APP --db "$DB"
```

## Segurança

- valide `SHA256SUMS` antes de executar downloads manuais;
- nunca edite o SQLite diretamente;
- mutações administrativas e applies exigem confirmação explícita;
- use `backlogctl version` e `backlogctl --json doctor --db PATH` no diagnóstico.

## Links

- [Última release](https://github.com/cadugevaerd/backlogctl-releases/releases/latest)
- [Plugin Claude](https://github.com/cadugevaerd/claude-skills/tree/main/plugins/backlog)
- [Plugin Codex](https://github.com/cadugevaerd/codex-skills/tree/main/plugins/backlog)
