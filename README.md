---
title: krprashantpk-plugins
description: "A GitHub Copilot agent-plugin marketplace for DevOps migration and backlog-to-code workflows."
---

A **GitHub Copilot agent-plugin marketplace** — a Git repository that contains a `marketplace.json` catalog describing one or more Copilot agent plugins. This repo is the catalog itself; consumers register it once and can then browse and install any plugin it lists.

> Status: shipping. The `plugins` array in [.github/plugin/marketplace.json](.github/plugin/marketplace.json) currently offers **code-shift** and **CodeStory**.

## Available plugins

| Plugin | Purpose |
| --- | --- |
| [code-shift](plugins/code-shift) | Migrates repositories and supporting project resources across Azure DevOps, GitHub, and GitLab. |
| [CodeStory](plugins/code-story) | Searches work items on GitHub, Jira Cloud, or Azure DevOps Services with chat-only results and optional comments/metadata; authors and exports stories, then supports planning, implementation, validation, and closeout. Bundles official hosted MCP connections for all three providers; see [setup and global-connection reuse](plugins/code-story/README.md#mcp-servers). |

## What is a Copilot plugin marketplace?

A marketplace is just a Git repo with a `marketplace.json` manifest at a recognized location (here: [.github/plugin/marketplace.json](.github/plugin/marketplace.json)). The manifest declares:

- `name` — the kebab-case registration key users type after `@` (here: `krprashantpk-plugins`).
- `owner` — who publishes the marketplace.
- `metadata` — human-readable description and marketplace version.
- `plugins` — the list of plugins the marketplace offers.

## Register this marketplace

### Copilot CLI

```sh
# Add the marketplace by its GitHub owner/repo
/plugin marketplace add krprashantpk/krprashantpk-plugins

# Browse the plugins it offers (by the marketplace name)
/plugin marketplace browse krprashantpk-plugins

# Install a specific plugin from this marketplace
/plugin install <plugin>@krprashantpk-plugins
```

### VS Code

1. Add `"krprashantpk/krprashantpk-plugins"` to the `chat.plugins.marketplaces` setting.
2. Open the Extensions view and browse `@agentPlugins` to discover and install plugins from registered marketplaces.

## Adding a new plugin

There are two ways to list a plugin in this marketplace. Each entry goes in the `plugins` array of [.github/plugin/marketplace.json](.github/plugin/marketplace.json).

### a) Monorepo (plugin lives in this repo)

Put the plugin under `plugins/<name>/` with its own `plugin.json`, then add an entry whose `source` is the **path inside this repo**:

```json
{
  "plugins": [
    {
      "name": "my-plugin",
      "version": "1.0.0",
      "source": "plugins/my-plugin"
    }
  ]
}
```

### b) External repo (plugin lives in a different repository)

Add an entry whose `source` is an **object** pointing at another repository:

```json
{
  "plugins": [
    {
      "name": "my-external-plugin",
      "version": "1.0.0",
      "source": {
        "source": "github",
        "repo": "krprashantpk/<other-repo>",
        "ref": "v1.0.0"
      }
    }
  ]
}
```

### `source`: string vs. object

- **String** → a path **inside this repo** (e.g. `"plugins/<name>"`).
- **Object** → a **different repo/URL** (e.g. `{ "source": "github", "repo": "krprashantpk/<other-repo>", "ref": "v1.0.0" }`).

## Publishing updates

Keep each plugin version synchronized between its `plugin.json` and [.github/plugin/marketplace.json](.github/plugin/marketplace.json). Consumers can pick up a published version with `/plugin update <plugin>`.

## Coding standards instructions

CodeStory packages all six shared instruction files in its [Copilot rules directory](plugins/code-story/com.github.copilot/rules). They are installed with the plugin rather than copied into a repository or user-profile instructions folder.

| File | Scope |
| --- | --- |
| [python.instructions.md](plugins/code-story/com.github.copilot/rules/python.instructions.md) | Python environments, dependencies, naming, types, models, docstrings, and inline comments. |
| [design-principal.instructions.md](plugins/code-story/com.github.copilot/rules/design-principal.instructions.md) | Focused responsibilities, explicit collaboration, practical reuse, and predictable behavior. |
| [environment-variables.instructions.md](plugins/code-story/com.github.copilot/rules/environment-variables.instructions.md) | Caller-supplied configuration, environment-variable ownership, and configured clients. |
| [logging-and-exceptions.instructions.md](plugins/code-story/com.github.copilot/rules/logging-and-exceptions.instructions.md) | Python logging, contextual messages, and protection of sensitive information. |
| [performance.instructions.md](plugins/code-story/com.github.copilot/rules/performance.instructions.md) | Non-blocking I/O, deadlock prevention, bounded concurrency, and explicit performance findings. |
| [documentation.instructions.md](plugins/code-story/com.github.copilot/rules/documentation.instructions.md) | Project setup documentation, Mermaid application flows, and package structure. |

CodeStory uses the Agent Plugins 1.0 manifest format. Skills and MCP servers use the portable `skills/` and `mcp.json` locations; instructions are Copilot-specific components under `com.github.copilot/rules/`. A compatible client and enabled plugin are required. The instruction contents and their existing `applyTo` patterns and descriptions are preserved; activation depends on those fields and the selected harness. See [rule verification](plugins/code-story/README.md#development-verification).

Repository-owned instructions remain a separate mechanism under `.github/instructions/`; the marketplace no longer keeps a second copy of these shared files there or in a root `instructions/` collection. Apply bundled rules alongside the target repository's instructions and surface conflicts rather than silently overriding either source.

## Project Structure

```
.
├── .github/
│   └── plugin/
│       └── marketplace.json                                      # Marketplace catalog manifest
├── plugins/
│   ├── code-shift/                                           # Cross-platform DevOps migration plugin
│   │   ├── plugin.json                                       # Code-Shift plugin manifest
│   │   ├── README.md                                         # Code-Shift usage and architecture
│   │   └── skills/
│   │       ├── migrate-branch-policies/SKILL.md              # Branch policy migration workflow
│   │       ├── migrate-issues/
│   │       │   ├── SKILL.md                                  # Issue and work-item migration workflow
│   │       │   └── references/github-to-ado-script-template.md # GitHub-to-Azure-DevOps script template
│   │       ├── migrate-pipelines/SKILL.md                    # CI/CD pipeline translation workflow
│   │       ├── migrate-pull-requests/SKILL.md                # Pull and merge request migration workflow
│   │       └── migrate-repos/
│   │           ├── SKILL.md                                  # Repository migration workflow
│   │           └── references/
│   │               ├── target-new-or-divergent.md            # New or divergent target handling
│   │               └── target-shared-history.md              # Shared-history target handling
│   └── code-story/                                           # Backlog-to-code workflow plugin
│       ├── plugin.json                                       # Agent Plugins 1.0 manifest
│       ├── mcp.json                                          # Hosted GitHub, Azure DevOps, and Atlassian MCP connections
│       ├── README.md                                         # CodeStory installation and usage
│       ├── com.github.copilot/
│       │   └── rules/                                       # Sole location for the six shared instruction files
│       └── skills/
│           ├── code-story-work-item-search/                  # Chat-only remote work-item search
│           │   ├── SKILL.md                                  # Shared provider-agnostic workflow
│           │   └── references/                              # Azure DevOps, GitHub, and Jira search guides
│           ├── code-story-author/SKILL.md                     # Story background authoring workflow
│           ├── code-story-export/SKILL.md                     # Confirmed CLI export and local URL write-back
│           ├── code-story-technical-plan/SKILL.md             # Technical planning workflow
│           └── code-story-implementer/SKILL.md                 # Implementation and closeout workflow
├── .gitignore                                                # Local ignore rules
├── ARCHITECTURE.md                                           # Marketplace and plugin architecture
├── LICENSE                                                   # MIT license
└── README.md                                                 # Marketplace setup and catalog
```

See [ARCHITECTURE.md](ARCHITECTURE.md) for marketplace discovery and plugin component interactions.

## License

[MIT](LICENSE)
