# Troubleshooting Professional Media over IP Networks

[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit)](https://github.com/pre-commit/pre-commit)
[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC_BY--NC--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

Use pandoc to convert to PowerPoint.

## Editor Setup

```bash
pip install pre-commit
npm install mermaid-filter
pre-commit install
pre-commit install --hook-type commit-msg
pandoc --citeproc --filter=node_modules/.bin/mermaid-filter.cmd \
  --from=markdown+yaml_metadata_block+emoji --slide-level=3 \
  .\troubleshooting-moip.md -o .\troubleshooting-moip.pptx
```
