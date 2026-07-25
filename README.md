# copilot-plugins

A **GitHub Copilot agent-plugin marketplace** — a Git repository that contains a `marketplace.json` catalog describing one or more Copilot agent plugins. This repo is the catalog itself; consumers register it once and can then browse and install any plugin it lists.

> Status: shipping. The `plugins` array in [.github/plugin/marketplace.json](.github/plugin/marketplace.json) currently offers one plugin: **code-shift** (see [plugins/code-shift](plugins/code-shift)).

## What is a Copilot plugin marketplace?

A marketplace is just a Git repo with a `marketplace.json` manifest at a recognized location (here: [.github/plugin/marketplace.json](.github/plugin/marketplace.json)). The manifest declares:

- `name` — the kebab-case registration key users type after `@` (here: `copilot-plugins`).
- `owner` — who publishes the marketplace.
- `metadata` — human-readable description and marketplace version.
- `plugins` — the list of plugins the marketplace offers (empty for now).

## Register this marketplace

### Copilot CLI

```sh
# Add the marketplace by its GitHub owner/repo
/plugin marketplace add krprashantpk/copilot-plugins

# Browse the plugins it offers (by the marketplace name)
/plugin marketplace browse copilot-plugins

# Install a specific plugin from this marketplace
/plugin install <plugin>@copilot-plugins
```

### VS Code

1. Add `"krprashantpk/copilot-plugins"` to the `chat.plugins.marketplaces` setting.
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

When you publish a change to a plugin, bump **both**:

1. The `version` on the plugin's entry in [.github/plugin/marketplace.json](.github/plugin/marketplace.json), and
2. The `version` in that plugin's own `plugin.json`.

Keeping the two in sync ensures consumers pick up the new release.

## Repository layout

```
.
├── .github/
│   └── plugin/
│       └── marketplace.json   # Marketplace catalog manifest (recognized location)
├── plugins/
│   └── code-shift/            # Any-direction repo/PR/policy/pipeline/issue migration plugin
├── .gitignore
├── LICENSE                    # MIT
└── README.md
```

## License

[MIT](LICENSE)
