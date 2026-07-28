# Proveniência dos artefatos — backlogctl v2.1.0

## Identidade

- release pública: [`v2.1.0`](https://github.com/cadugevaerd/backlogctl-releases/releases/tag/v2.1.0)
- versão do binário: `2.1.0`
- schema SQLite: `5`
- commit privado usado para os builds: `c12391e4e90349434c79fc483d865423e2ff17e1`
- `main` privado verificado após o build: `7c78626f78abe1d2c7037cb318e2256fc8244640`

## Relação entre build e main

O único commit posterior ao build adiciona `.github/workflows/ci.yml` e torna testes black-box portáveis. Não altera arquivos de produção, migrations, contratos da CLI ou o processo de build. Por isso, não houve reconstrução nem substituição dos assets `v2.1.0`.

## Verificação

- GitHub Actions: [`quality` e `race` em sucesso](https://github.com/cadugevaerd/backlogctl/actions/runs/30367434937)
- `quality`: gofmt, `go test -count=1`, `go vet` e `go build`
- `race`: `CGO_ENABLED=1 go test -race -count=1 -timeout 300s ./...`
- seis hashes SHA-256 estão em [`SHA256SUMS`](SHA256SUMS)
- o mapeamento plataforma/asset está em [`backlogctl-release.json`](backlogctl-release.json)

As cópias no branch `main` são metadados auditáveis. Os binários canônicos continuam sendo os assets anexados à release `v2.1.0`.
