<!-- README.md -->
# 👁️ Code Literal Sentinel

[![GitHub Release](https://img.shields.io/github/v/release/Malnati/code-literal-sentinel?style=for-the-badge&color=orange)](https://github.com/Malnati/code-literal-sentinel/releases)

**Varredura agressiva de literais hardcoded para indexação e análise assistida por IA.**

---

## 🔐 Configuração Crítica de Segurança (Obrigatório)

Para que esta Action possa criar a Pull Request com o relatório, você **precisa** autorizar o GitHub Actions no seu repositório. Sem isso, você receberá erros de `403 Resource not accessible`.

1.  No seu repositório, vá em **Settings**.
2.  Na barra lateral, expanda o menu **Actions** e clique em **General**.
3.  Role até o final, na seção **Workflow permissions**.
4.  Marque a opção: `Allow GitHub Actions to create and approve pull requests`.
5.  Clique em **Save**.



---

## 🚀 Como Funciona

1.  **Scan:** Busca agressiva (grep) por strings e números mágicos no diff da PR.
2.  **Isolamento:** Cria uma branch dedicada `sentinel/audit/<sua-branch>`.
3.  **Relatório:** Gera um arquivo Markdown com as evidências encontradas.
4.  **Entrega:** Abre uma PR automática contendo apenas este relatório, facilitando a revisão ou o consumo por agentes de IA.

---

## 📦 Inputs

| Input | Descrição | Padrão |
| :--- | :--- | :--- |
| `token` | **Obrigatório**. Token para git push e criação de PR. | - |
| `file_extensions` | Extensões alvo. | `ts\|js\|java...` |
| `exclude_patterns` | Padrões ignorados. | `node_modules\|dist...` |
| `report_dir` | Onde salvar o relatório. | `reports/code-literal-sentinel` |

---

## 🛠️ Exemplo de Workflow

```yaml
name: "Sentinel Scan"
on: [pull_request]

permissions:
  contents: write       # Necessário para push do relatório
  pull-requests: write  # Necessário para abrir a PR de auditoria

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # 1. Roda o Sentinel
      - name: Run Scanner
        id: sentinel
        uses: Malnati/code-literal-sentinel@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      # 2. Notifica na PR original
      - name: Notify User
        uses: Malnati/pr-comment@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: ${{ github.event.pull_request.number }}
          message_id: "sentinel-report"
          header_title: "👁️ Code Literal Sentinel"
          header_subject: "Relatório de Auditoria"
          header_actor: "github-actions[bot]"
          
          # Usa os outputs ricos do Sentinel v2
          body_message: |
            ### ${{ fromJson(steps.sentinel.outputs.result_json).ui.message }}
            ${{ fromJson(steps.sentinel.outputs.result_json).ui.guidance }}
          
          footer_result: ${{ fromJson(steps.sentinel.outputs.result_json).analysis.status == 'FOUND' && '⚠️ Revisão' || '✅ Limpo' }}
          footer_advise: "Siga o link acima para ver os detalhes."
