# Proveniência dos artefatos — backlogctl v2.4.0

## Identidade

- release pública: [`v2.4.0`](https://github.com/cadugevaerd/backlogctl-releases/releases/tag/v2.4.0)
- versão do binário: `2.4.0`
- schema SQLite: `5`
- commit privado usado para os builds: `5ffe1dfabe26b2756f1df5388a5fda4b9daae5b7`
- `main` privado no momento do build: o **mesmo** commit; não há delta entre build e `main`

## O que mudou de comportamento

`doctor` é read-only. Ele abria o banco criando o arquivo ausente e chamava `Init()`, que aplica migrations: num caminho inexistente respondia `schema: 5, integrity_check: ok` sobre um arquivo que não existia; num store atrasado, migrava e relatava a versão que acabou de produzir. Eram **duas portas para a mesma mutação irreversível** — `update migrate` exige `--backup-dir`, `--confirm` e verificação de integridade; o `doctor` chegava ao mesmo `Init()` sem nenhum dos três.

Agora `doctor` usa `OpenExisting` e não migra. Num caminho que não existe ele **falha**, nomeando o caminho, em vez de criar o banco e aprovar.

Sai como MINOR e não MAJOR porque o comportamento removido violava o Princípio II da constituição do projeto, e remover violação é correção.

Também nesta versão: `doctor` sobre um SQLite que não é store deste produto diz isso, em vez de devolver `no such table: schema_migrations`; e as cinco mensagens de Vínculo passam a nomear o caminho declarado, o backlog envolvido e a causa de origem.

Nenhum contrato mudou: schema `5`, envelope `contract_version: "2"`, documento de import `"3"`.

## Verificação

- workflow de release: [suíte macOS e os seis cross-builds em sucesso](https://github.com/cadugevaerd/backlogctl/actions/runs/31430619903). A suíte macOS é pré-requisito declarado dos builds: falha nela impede qualquer artefato de existir.
- os seis assets foram conferidos **baixados da release**, hash a hash, contra [`backlogctl-release.json`](backlogctl-release.json): 6 de 6 conferem.
- reprodutibilidade verificada: um clone limpo na tag `v2.4.0`, com `core.autocrlf=false`, regenera os seis binários com hash idêntico ao publicado.
- seis hashes SHA-256 estão em [`SHA256SUMS`](SHA256SUMS); o mapeamento plataforma/asset está em [`backlogctl-release.json`](backlogctl-release.json).

### Ressalva honesta sobre o CI no commit de build

O workflow `quality` estava **vermelho** neste commit, por dois motivos que não afetam os artefatos, e vale registrar em vez de omitir:

1. em `ubuntu-latest`, o portão de cobertura reprovou com `cobertura 80.8 / piso 80.9`. O piso havia sido elevado a partir de uma medição em Windows, que lê ~0,1 ponto mais alto do que a de Linux, onde o portão é cobrado;
2. em `windows-latest`, um teste que lê o próprio fonte em tempo de execução falhou porque o checkout do CI em Windows usa CRLF por padrão.

Nenhum dos dois é comportamento de produto: os testes passaram em `ubuntu-latest`, a suíte de macOS passou no workflow de release, e os binários são construídos em Linux com LF. Os dois foram corrigidos em commit posterior no repositório privado — o segundo por um `.gitattributes` que fixa `eol=lf` para fonte Go, que é também o que garante a reprodutibilidade acima.

## Semântica das tags deste repositório

A tag aqui é a âncora do objeto de release, não a descrição dos metadados: ela aponta para o `main` que existia quando a release foi criada. Vale para todas as versões — `v2.1.0`, por exemplo, aponta para o commit que documentava a v2.0.2. Para saber a que versão um commit deste `main` corresponde, leia o bloco "Release atual" do [README](README.md).

As cópias no branch `main` são metadados auditáveis. Os binários canônicos continuam sendo os assets anexados à release `v2.4.0`.
