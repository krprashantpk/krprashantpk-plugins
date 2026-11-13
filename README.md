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

Repository-wide performance and async safety rules are automatically applied
through [performance.instructions.md](.github/instructions/performance.instructions.md).
They require non-blocking I/O wherever supported, deadlock prevention, bounded
concurrency, and explicit flags for inefficient code encountered during a task.

The [instructions](instructions) folder contains reusable coding standards:

| File | Scope |
| --- | --- |
| [python.instructions.md](instructions/python.instructions.md) | Python runtimes, dependencies, formatting, naming, types, functions, classes, imports, docstrings, and inline comments. |
| [design-principal.instructions.md](instructions/design-principal.instructions.md) | Focused responsibilities, explicit collaboration, practical reuse, and predictable behavior. |
| [configuration.instructions.md](instructions/configuration.instructions.md) | Explicit settings, validation, configured clients, and ownership boundaries. |
| [architecture.instructions.md](instructions/architecture.instructions.md) | Package layout, component contracts, dependency direction, data integrity, and compatibility. |
| [logging-and-exceptions.instructions.md](instructions/logging-and-exceptions.instructions.md) | Contextual Python logging, validation errors, exception propagation, and explicit failure outcomes. |
| [documentation.instructions.md](instructions/documentation.instructions.md) | Root project documents, package and application README sections, Mermaid application flows, and package structure. |

These files are a reusable collection, not installed plugin content. The root
`instructions/` folder is not a default workspace instruction-discovery location.
To use them, copy the selected files into a target workspace's
`.github/instructions/` folder or a VS Code user-profile instructions location.
Some instructions include file-matching `applyTo` patterns or task-oriented
descriptions so they can be loaded when relevant or attached explicitly. Add
repository-specific path patterns where needed.

Python docstrings and inline comments belong only in the Python instructions.
Documentation instructions are for project documentation, not docstrings.
The [standards mapping](brainstorm/common-standard.md) tracks the broader plan;
the remaining instruction files have not been created.

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
│       ├── plugin.json                                       # CodeStory plugin manifest
│       ├── .mcp.json                                         # Hosted GitHub, Azure DevOps, and Atlassian MCP connections
│       ├── README.md                                         # CodeStory installation and usage
│       └── skills/
│           ├── code-story-work-item-search/                  # Chat-only remote work-item search
│           │   ├── SKILL.md                                  # Shared provider-agnostic workflow
│           │   └── references/                              # Azure DevOps, GitHub, and Jira search guides
│           ├── code-story-author/SKILL.md                     # Story background authoring workflow
│           ├── code-story-export/SKILL.md                     # Confirmed CLI export and local URL write-back
│           ├── code-story-technical-plan/SKILL.md             # Technical planning workflow
│           └── code-story-implementer/SKILL.md                 # Implementation and closeout workflow
├── .gitignore                                                # Local ignore rules
├── architecture.md                                          # Marketplace and plugin architecture
├── LICENSE                                                   # MIT license
└── README.md                                                 # Marketplace setup and catalog
```

## License

[MIT](LICENSE)
