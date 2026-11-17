# Project Structure

This repository contains a Copilot plugin marketplace and independently installable plugins.

```text
.
|-- .github/
|   `-- plugin/
|       `-- marketplace.json
|-- plugins/
|   |-- code-shift/
|   |   |-- plugin.json
|   |   |-- README.md
|   |   `-- skills/
|   |-- code-story/
|   |   |-- plugin.json
|   |   |-- mcp.json
|   |   |-- README.md
|   |   |-- com.github.copilot/
|   |   |   `-- rules/
|   |   `-- skills/
|   `-- cu-studio/
|       |-- plugin.json
|       |-- README.md
|       `-- skills/
|           |-- edit-analyzer-json/
|           |   |-- SKILL.md
|           |   `-- references/
|           `-- verify-document-extraction/
|               `-- SKILL.md
|-- .backlog/
|-- ARCHITECTURE.md
|-- LICENSE
|-- PROJECT_STRUCTURE.md
`-- README.md
```

| Path | Responsibility |
| --- | --- |
| [.github/plugin/marketplace.json](.github/plugin/marketplace.json) | Catalog entries with plugin names, versions, descriptions, and sources. |
| [plugins/code-shift/](plugins/code-shift) | Cross-platform DevOps migration skills and their provider references. |
| [plugins/code-story/](plugins/code-story) | Work-item search, story authoring and export, technical planning, implementation, and review workflows. |
| [plugins/code-story/com.github.copilot/rules/](plugins/code-story/com.github.copilot/rules) | Shared coding instructions packaged with CodeStory. |
| [plugins/code-story/mcp.json](plugins/code-story/mcp.json) | Hosted MCP definitions bundled with CodeStory; authentication remains client-owned. |
| [plugins/cu-studio/](plugins/cu-studio) | Azure AI Content Understanding plugin with its manifest, setup, and authoring documentation. |
| [plugins/cu-studio/skills/edit-analyzer-json/](plugins/cu-studio/skills/edit-analyzer-json) | Analyzer JSON editing workflow and a supporting best-practices reference. |
| [plugins/cu-studio/skills/verify-document-extraction/](plugins/cu-studio/skills/verify-document-extraction) | Chat-only comparison of CU raw results with source PDFs, without business-rule validation. |
| [.backlog/](.backlog) | Local work items and technical plans; not shipped as plugin components. |
| [README.md](README.md) | Marketplace registration, installation, and publishing guidance. |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Marketplace discovery and implemented plugin interactions. |

Keep each plugin's components within its own directory. Each skill lives in `skills/<skill-name>/SKILL.md`, with optional supporting references or scripts alongside it. Match every plugin's manifest name and version to its marketplace entry.
