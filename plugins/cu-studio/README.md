# CU Studio

A GitHub Copilot agent plugin for Azure AI Content Understanding (CU) skills.

Includes skills for editing analyzer JSON and verifying extraction results against source PDFs. No MCP servers or Azure connections are bundled. Installing the plugin does not provision resources or call Azure.

## Prerequisites

| Tool or service | Required version or access | Purpose |
| --- | --- | --- |
| GitHub Copilot | A plugin-capable Copilot CLI or VS Code client | Install the plugin and discover skills as they are added. |
| Git | Required only for local development | Clone and edit the plugin source. |
| Microsoft documentation MCP tools | Required by the analyzer-editing workflow; configured separately | Research current CU schemas, limits, and best practices. |
| Local PDF reading/rendering tools | Required for extraction verification; OCR when needed | Compare the source PDF with CU results without uploading source artifacts. |

Local editing and comparison do not require Azure credentials. Deployment, when explicitly requested, requires the consuming project's deployment scripts, configuration, and Azure access.

## Local setup

1. Clone `krprashantpk/krprashantpk-plugins` and open the repository.
2. Edit this plugin under `plugins/cu-studio/`. The plugin itself has no dependency installation or supporting services; individual workflows require the tools listed above.
3. Register the marketplace and install the plugin in Copilot CLI:

   ```text
   /plugin marketplace add krprashantpk/krprashantpk-plugins
   /plugin install cu-studio@krprashantpk-plugins
   ```

Marketplace installation uses the published repository, not unpublished local edits. Commit and publish changes before expecting other clients to install them.

For VS Code installation, follow the [marketplace setup](../../README.md#vs-code).

## Environment variables

The plugin defines no environment variables and stores no credentials. For optional deployment, follow the consuming project's configuration and authentication requirements.

## Running locally

There is no application or local server to start. After installation, verify that `cu-studio` and its two skills appear in the client's customizations, then request the appropriate workflow:

- "Edit this Azure Content Understanding analyzer JSON."
- "Verify this CU raw-result JSON against its source PDF."

After publishing an update, run `/plugin update cu-studio` and start a new session.

## Included skills

| Skill | Purpose |
| --- | --- |
| [edit-analyzer-json](skills/edit-analyzer-json/SKILL.md) | Research CU guidance, edit analyzer fields/categories/configuration, and validate JSON; deploy only if requested. Includes a [best-practices reference](skills/edit-analyzer-json/references/best-practices.md). |
| [verify-document-extraction](skills/verify-document-extraction/SKILL.md) | Compare a CU raw-result JSON with its source PDF, including transcription, structured values, associations, and available grounding. Reports findings in chat, without applying business rules or changing inputs. |

These workflows retain consuming-project assumptions. The analyzer editor references `scripts/analyzers/`, deployment/validation PowerShell scripts, and application-specific SDK/model conventions. The verifier includes a pre-execution check for `src/final_invoice/local.settings.json` under its source repository's rules. Those project files are not bundled here; confirm the target project's equivalent context and resolve inapplicable instructions before running the affected steps. Never create or expose credentials merely to satisfy a missing reference.

## Running tests

No automated tests or build tooling are bundled. Before publishing, validate the manifest JSON against its declared schema, ensure its name and version match the [marketplace entry](../../.github/plugin/marketplace.json), check skill frontmatter and reference links, and verify discovery in a plugin-capable client. End-to-end workflow checks require the relevant project context and sample inputs; packaging checks alone do not verify Azure behavior or extraction accuracy.

## Adding skills

Create one directory per skill under [skills/](skills), with a `SKILL.md` file containing the required discovery frontmatter and workflow instructions:

```text
skills/
    <skill-name>/
        SKILL.md
        references/    (optional)
        scripts/       (optional)
```

Use descriptive, kebab-case skill names. Document clear activation conditions, required inputs, authentication, expected outputs, error handling, and any confirmation required before Azure writes. Do not put credentials or service-specific configuration into the plugin manifest.

The Agent Plugins 1.0 manifest uses the conventional `skills/` discovery location. Keep the plugin and marketplace versions synchronized when publishing.

See [project structure](../../PROJECT_STRUCTURE.md) for component boundaries and the [marketplace architecture](../../ARCHITECTURE.md) for discovery flow.

## License

[MIT](../../LICENSE)
