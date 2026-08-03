# krprashantpk-plugins

A **GitHub Copilot agent-plugin marketplace** — a Git repository that contains a `marketplace.json` catalog describing one or more Copilot agent plugins. This repo is the catalog itself; consumers register it once and can then browse and install any plugin it lists.

> Status: shipping. The `plugins` array in [.github/plugin/marketplace.json](.github/plugin/marketplace.json) currently offers **code-shift** and **CodeStory**.

## Available plugins

| Plugin | Purpose |
| --- | --- |
| [code-shift](plugins/code-shift) | Migrates repositories and supporting project resources across Azure DevOps, GitHub, and GitLab. |
| [CodeStory](plugins/code-story) | Moves backlog ideas through story authoring, technical planning, implementation, validation, and closeout. |

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
│       ├── README.md                                         # CodeStory installation and usage
│       └── skills/
│           ├── backlog-story-author/SKILL.md                 # Story background authoring workflow
│           ├── backlog-story-technical-plan/SKILL.md         # Technical planning workflow
│           └── backlog-story-implementer/
│               ├── SKILL.md                                  # Implementation and closeout workflow
│               └── references/python-and-project-conventions.md # Python and project implementation defaults
├── .gitignore                                                # Local ignore rules
├── architecture.md                                          # Marketplace and plugin architecture
├── LICENSE                                                   # MIT license
└── README.md                                                 # Marketplace setup and catalog
```

## License

[MIT](LICENSE)
