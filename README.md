# backlogctl — releases públicas

`backlogctl` é uma CLI local-first para **operacionalizar qualquer tipo de trabalho**: vida pessoal, rotinas operacionais, projetos técnicos, produtos, estudos, manutenção ou demandas de equipes.

Ela mantém **múltiplos backlogs independentes em paralelo** no mesmo banco SQLite. Cada backlog reúne seus próprios itens, prioridades, estados, posição, **descrições executáveis**, contexto de priorização e decisões auditáveis.

Este repositório **não contém o código-fonte**. Ele distribui somente binários públicos e verificáveis usados pelos plugins oficiais. Cada release contém `SHA256SUMS` para validar o binário antes de executá-lo.

## O que a CLI faz

```text
Backlogs        criar e operar vários backlogs independentes em paralelo: pessoal, operacional, técnico, produto, estudos ou qualquer outro contexto; listar, editar, arquivar e vincular a um path
Itens           criar, listar, consultar, editar, transicionar, reconciliar status, arquivar e reordenar; cada item tem título-resumo e descrição executável
Prioridades     critical / high / medium / low, com posição dentro da faixa
Contextos       registrar sinais de prioridade com validade, revisão e expiração
Format           propor reorganização e aplicar somente após confirmação explícita
Export           JSON, Markdown e visão consolidada
Doctor           inicializar/verificar o banco e a versão do schema
```

### Exemplos de backlogs paralelos

```text
PES         vida, finanças, saúde, compras e pendências
OPS         rotinas, incidentes, fornecedores e checklist recorrente
PRD         melhorias, bugs, decisões e entregas
EST         trilhas, práticas e metas de aprendizagem
```

Eles coexistem no mesmo banco, mas possuem códigos, itens, vínculo de path e ordenação próprios.

A CLI usa um banco indicado por `--db PATH`; ela não exige um banco global nem acesso direto ao SQLite por agentes.

> **Migração V1:** o plugin `backlog` lê o JSON V1 inteiro, gera uma proposta humana revisável e só então chama esta CLI para criar os dados V2. Não existe um comando `import` implícito nem alteração automática do JSON legado.

## Instalação recomendada: plugin público

O plugin cuida do download, da seleção de plataforma, da verificação SHA-256 e da instalação atômica do binário.

### Claude Code

```bash
claude plugin marketplace add cadugevaerd/claude-skills
claude plugin install backlog@claude-skills
```

Ao iniciar uma sessão, o plugin instala ou reutiliza automaticamente um `backlogctl` verificado. O hook informa ao agente o caminho exato do executável; não é necessário adicioná-lo ao `PATH`.

- Plugin: <https://github.com/cadugevaerd/claude-skills/tree/main/plugins/backlog>
- Marketplace: <https://github.com/cadugevaerd/claude-skills>

### Codex

```bash
git clone https://github.com/cadugevaerd/codex-skills.git
cd codex-skills
codex plugin marketplace add .
codex plugin add backlog@codex-skills
```

No Codex, execute a recuperação/bootstrap quando necessário:

```bash
node plugins/backlog/scripts/ensure-backlogctl.js --install-dir "$HOME/.local/bin"
```

- Plugin: <https://github.com/cadugevaerd/codex-skills/tree/main/plugins/backlog>

## Instalação manual

Use a [release mais recente](https://github.com/cadugevaerd/backlogctl-releases/releases/latest). Escolha o arquivo para seu sistema:

| Plataforma | Arquivo |
|---|---|
| Linux x86_64 | `backlogctl_linux_amd64` |
| Linux ARM64 | `backlogctl_linux_arm64` |
| macOS Intel | `backlogctl_darwin_amd64` |
| macOS Apple Silicon | `backlogctl_darwin_arm64` |
| Windows x86_64 | `backlogctl_windows_amd64.exe` |
| Windows ARM64 | `backlogctl_windows_arm64.exe` |

### Linux ARM64 — exemplo completo

```bash
version=v2.0.2
base="https://github.com/cadugevaerd/backlogctl-releases/releases/download/${version}"
curl -fL -O "$base/backlogctl_linux_arm64"
curl -fL -O "$base/SHA256SUMS"
grep '  backlogctl_linux_arm64$' SHA256SUMS | sha256sum -c -
mkdir -p "$HOME/.local/bin"
install -m 0755 backlogctl_linux_arm64 "$HOME/.local/bin/backlogctl"
"$HOME/.local/bin/backlogctl" version
```

Para Linux x86_64, troque `linux_arm64` por `linux_amd64`. Em macOS, use o binário `darwin_*` e valide com `shasum -a 256 -c SHA256SUMS`.

### Windows PowerShell — exemplo completo

```powershell
$version = 'v2.0.2'
$asset = 'backlogctl_windows_amd64.exe'
$base = "https://github.com/cadugevaerd/backlogctl-releases/releases/download/$version"
Invoke-WebRequest "$base/$asset" -OutFile $asset
Invoke-WebRequest "$base/SHA256SUMS" -OutFile SHA256SUMS
$expected = ((Get-Content SHA256SUMS | Select-String "  $asset`$").ToString() -split '\s+')[0].ToLower()
$actual = (Get-FileHash $asset -Algorithm SHA256).Hash.ToLower()
if ($actual -ne $expected) { throw 'SHA-256 inválido; binário não será instalado.' }
New-Item -ItemType Directory -Force "$env:LOCALAPPDATA\backlogctl\bin" | Out-Null
Move-Item $asset "$env:LOCALAPPDATA\backlogctl\bin\backlogctl.exe" -Force
& "$env:LOCALAPPDATA\backlogctl\bin\backlogctl.exe" version
```

## Primeiros comandos

Use um caminho explícito para o banco. Em scripts/agentes, `--json` vem **antes** da família de comandos.

```bash
BACKLOGCTL="$HOME/.local/bin/backlogctl"
DB="$HOME/.backlog/backlog.db"

# prepara e diagnostica o banco
"$BACKLOGCTL" store init --db "$DB"
"$BACKLOGCTL" --json doctor --db "$DB"

# cria um backlog e um item
"$BACKLOGCTL" --json backlog create \
  --code APP --name "Aplicação principal" --profile software --db "$DB"
"$BACKLOGCTL" --json item add \
  --code APP --title "Corrigir timeout de webhook" \
  --description "Revisar timeout, adicionar retry e validar o fluxo de entrega." \
  --criticality high --category bug --db "$DB"

# para uma migração confirmada, --status cria o snapshot no estado informado
"$BACKLOGCTL" --json item add \
  --code APP --title "Incidente histórico encerrado" \
  --description "Registro preservado da fonte de origem." \
  --status done --criticality medium --category general --db "$DB"

# correções verificadas e itens de teste usam operações auditáveis, nunca SQL direto
"$BACKLOGCTL" --json item reconcile-status \
  --id APP-2 --status cancelled --reason "Correção validada" --confirm --db "$DB"
"$BACKLOGCTL" --json item archive \
  --id APP-2 --reason "Item de teste sem fonte" --confirm --db "$DB"

# flags pertencem ao comando: uso incompatível falha com exit 2
# item transition usa --status; merged permanece terminal nesse fluxo normal

# edita somente o descritivo; omitir --description preserva o texto atual
"$BACKLOGCTL" --json item edit \
  --id APP-1 --description "Timeout revisado; falta validar a integração." --db "$DB"

# consulta a fila e exporta um snapshot
"$BACKLOGCTL" --json item list --code APP --db "$DB"
"$BACKLOGCTL" --json export consolidated --code APP --db "$DB"
```

## Segurança e contrato

- Baixe somente artefatos de uma release publicada e valide o SHA-256 antes de executar.
- Não edite o SQLite diretamente; use a CLI.
- Comandos mutáveis de `context` e `format apply` exigem confirmação humana explícita no workflow do plugin.
- `format propose` é somente proposta: não muda os itens. `format apply --confirm` rejeita propostas expiradas ou obsoletas.
- Use `backlogctl version` e `backlogctl --json doctor --db PATH` ao abrir um incidente.

## Releases

- [Última release](https://github.com/cadugevaerd/backlogctl-releases/releases/latest)
- [Todas as releases](https://github.com/cadugevaerd/backlogctl-releases/releases)
- [Plugin público Claude Code](https://github.com/cadugevaerd/claude-skills/tree/main/plugins/backlog)
- [Plugin público Codex](https://github.com/cadugevaerd/codex-skills/tree/main/plugins/backlog)
