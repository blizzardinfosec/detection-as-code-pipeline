# 🚦 Detection-as-Code Pipeline

This project demonstrates a basic CI/CD pipeline for Detection-as-Code using Sigma. It automatically:

- ✅ Validates all Sigma rules via linter
- 🔄 Converts Sigma to SIEM-compatible queries (e.g., Splunk, Elastic)
- 📦 Outputs ready-to-deploy rules

## 🧰 Tools Used
- GitHub Actions
- [sigmac](https://github.com/SigmaHQ/sigma)
- Python 3.11+
- YAML linting

## 📁 Structure
/rules/ → Source Sigma rules
/converted/ → Transformed rules for backend use (e.g., SPL, KQL)
/.github/workflows/ci.yml → CI pipeline to validate and convert


---

### ✅ GitHub Action: `.github/workflows/ci.yml`

Create a folder `.github/workflows/` and inside it, a file `ci.yml`:

```yaml
name: Sigma CI Pipeline

on:
  push:
    paths:
      - 'rules/**.yml'
      - '.github/workflows/ci.yml'
  pull_request:
    paths:
      - 'rules/**.yml'

jobs:
  validate-and-convert:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install pyyaml
          git clone https://github.com/SigmaHQ/sigma
          cd sigma/tools && pip install -r requirements.txt

      - name: Validate Sigma rules
        run: |
          cd sigma/tools
          for file in ../../rules/*.yml; do
            python sigmac --target splunk -c ../config/splunk-windows.yml "$file" > /dev/null || {
              echo "Failed to parse $file"
              exit 1
            }
          done

      - name: Convert rules to Splunk format
        run: |
          mkdir -p converted
          cd sigma/tools
          for file in ../../rules/*.yml; do
            out=../../converted/$(basename "$file" .yml).spl
            python sigmac --target splunk -c ../config/splunk-windows.yml "$file" > "$out"
          done
