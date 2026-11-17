# krprashantpk-plugins

A collection of GitHub Copilot agent plugins for DevOps migration, backlog-to-code workflows, and Azure AI Content Understanding. Register this marketplace once, then install only the plugins you need.

## Available Plugins

| Plugin | Purpose |
| --- | --- |
| [code-shift](plugins/code-shift/README.md) | Migrate repositories, pull requests, branch policies, pipelines, and issues across Azure DevOps, GitHub, and GitLab. |
| [CodeStory](plugins/code-story/README.md) | Search work items, author and export stories, approve technical plans, and implement and verify changes. |
| [CU Studio](plugins/cu-studio/README.md) | Edit Azure AI Content Understanding analyzer JSON and verify extraction results against source PDFs. |

## Prerequisites

| Tool or service | Required version or access | Purpose |
| --- | --- | --- |
| GitHub Copilot CLI or VS Code | A plugin-capable client with Copilot access | Install and use the plugins. |
| Plugin-specific tools and services | As documented in each plugin's README above | Run the selected workflows and authenticate to their providers. |

## Local Setup

### Copilot CLI

Register the marketplace from your terminal:

```sh
copilot plugin marketplace add krprashantpk/krprashantpk-plugins
```

Optionally browse the catalog before installing:

```sh
copilot plugin marketplace browse krprashantpk-plugins
```

### VS Code

1. Add `"krprashantpk/krprashantpk-plugins"` to the `chat.plugins.marketplaces` setting.
2. Open the Extensions view and search for `@agentPlugins`.
3. Select and install the plugins you want.

## Install Plugins

For Copilot CLI, run any of the following terminal commands after registering the marketplace. VS Code users can install directly from the Extensions view.

### code-shift

```sh
copilot plugin install code-shift@krprashantpk-plugins
```

### CodeStory

```sh
copilot plugin install code-story@krprashantpk-plugins
```

### CU Studio

```sh
copilot plugin install cu-studio@krprashantpk-plugins
```

## Documentation

- [Project structure](PROJECT_STRUCTURE.md): plugin layout and component responsibilities.s
- [Marketplace manifest](.github/plugin/marketplace.json): plugin names, versions, and sources.
- [CodeStory MCP setup](plugins/code-story/README.md#mcp-servers): hosted connections, authentication, and reuse of global connections.
