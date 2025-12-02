<!-- README.md -->
# hardcoded-check

Ação reutilizável do GitHub Actions que gera um índice de literais hardcoded a partir dos arquivos alterados em uma Pull Request e publica um resumo como comentário no próprio PR. Suporta arquivos `.ts`, `.js`, `.jsx`, `.tsx`, `.java`, `.py`, `.sh` e `.bash`.

## Uso

Adicione no repositório de aplicação um workflow de PR que chame o `hardcoded-check` após o `auto-sync`:

```yaml
# .github/workflows/pr-flow.yml
name: "PR Flow"

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  flow-check:
    uses: Malnati/flow-check/.github/workflows/flow-check.yml@v1

  auto-sync:
    needs: flow-check
    uses: Malnati/auto-sync/.github/workflows/auto-sync.yml@v1
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}

  hardcoded-check:
    needs: auto-sync
    uses: Malnati/hardcoded-check/.github/workflows/hardcoded-check.yml@v1
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
