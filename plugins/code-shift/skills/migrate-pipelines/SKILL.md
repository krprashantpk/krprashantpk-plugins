---
name: migrate-pipelines
description: 'Migrate CI/CD pipeline definitions across Azure DevOps (azure-pipelines.yml and Classic), GitHub Actions, and GitLab CI in any direction by translating the pipeline YAML between platforms. Use when moving pipelines, workflows, or CI/CD definitions between these platforms (for example "convert the Azure Pipelines YAML to GitHub Actions", "translate .gitlab-ci.yml for Azure DevOps"). Self-contained: always runs a read-only dry-run first, migrates only what the user selected, confirms every destructive action, and never uses Personal Access Tokens. Translation is required, not a copy, and translated files must be reviewed before running. Not for repositories, pull requests, issues, or branch policies.'
license: MIT
---

# Migrate `pipes` — CI/CD pipeline definitions

Move CI/CD pipeline definitions across Azure DevOps, GitHub Actions, and GitLab CI, in any direction. Pipeline definitions travel inside the repo as YAML but use **different syntax per platform** — translate them, never blind-copy. Always mark translated files as **review required** before they run.

## Operating rules — never break these

1. **Dry-run first, always.** Begin with a read-only preview that touches nothing on the target: read the source pipeline definitions, plan the translation, and write out exactly what would be created, changed, skipped, or cannot migrate. Proceed only on explicit user approval.
2. **Only what the user selected.** This skill migrates pipeline definitions (`pipes`). Never expand into repositories, pull requests, issues, or policies without asking.
3. **Confirm every destructive action.** Report each write before performing it and get explicit confirmation. Never silently overwrite an existing pipeline file — that is an override and needs its own second, explicit confirmation.

**No Personal Access Tokens.** Reuse an existing signed-in session and verify both source and target are authenticated before doing anything:

| Platform | Verify | Sign in |
| --- | --- | --- |
| Azure DevOps | `az account show` | `az login` |
| GitHub | `gh auth status` | `gh auth login` |
| GitLab | `glab auth status` | `glab auth login` |

If either side is not signed in, stop and ask the user to authenticate. Prefer the platform **CLI (with `*/api` REST passthrough) for writes** and the **MCP server for read/preview** — `code-shift-github`, `code-shift-azure-devops`, `code-shift-gitlab` (see `mcp.json`).

## Workflow

1. **Preflight** — confirm the source and target coordinates and that both are authenticated. Pipeline files live in the repo (migrate the repository first, or read the source files directly).
2. **Discover (read-only)** — locate the source pipeline definitions and inspect the target for existing pipeline files (see commands below).
3. **Dry-run report** — write out each pipeline that would be translated to the target syntax, what cannot migrate cleanly (ADO Classic, unmapped tasks/actions), and the secret **names** the pipeline needs. Nothing is written. Stop for explicit approval.
4. **Confirm & execute** — after approval, confirm committing each translated file, then commit it. On an existing pipeline file, offer **abort** or **override**; an override needs a second explicit confirmation.
5. **Verify & summarize** — confirm the translated files exist and remind the user to review them and re-enter secret values.

## Syntax mapping (where each platform expects its pipeline)

| Platform | Pipeline file(s) | Engine |
| --- | --- | --- |
| Azure DevOps | `azure-pipelines.yml` (YAML) | Azure Pipelines |
| GitHub | `.github/workflows/*.yml` | GitHub Actions |
| GitLab | `.gitlab-ci.yml` | GitLab CI/CD |

Concept translation (approximate): stages/jobs/steps ⇄ jobs/steps (Actions) ⇄ stages/jobs (GitLab); triggers (`trigger:`/`pr:`) ⇄ `on:` ⇄ `rules:`/`workflow:`; variables ⇄ `env:` ⇄ `variables:`; ADO tasks ⇄ Actions `uses:` ⇄ GitLab `script:` shell steps.

## Azure DevOps Classic (no YAML)

Classic (UI-designed) build/release definitions have **no YAML**. Export via REST and synthesize an equivalent on the target:

```bash
az devops invoke --area build --resource definitions --route-parameters project=<p> --http-method GET
```

Flag all Classic-derived pipelines as **needs human review** — the synthesis is best-effort.

## Discover (read-only)

- Detect pipeline files in the source repo (they came over with the repository, or read them directly).
- Azure DevOps: `az pipelines list`; GitHub: `gh workflow list`; GitLab: `glab ci list` / read `.gitlab-ci.yml`.

## Dry-run entries

- Will create: N pipeline files, **translated** to the target syntax (name each).
- Cannot migrate cleanly: ADO Classic definitions (exported + synthesized), platform-specific tasks/marketplace actions with no equivalent.
- Manual review: every translated file must be reviewed before enabling.

## Execute (confirm each write — rule 3)

1. Translate each pipeline to the target's syntax and file location.
2. **Confirm** committing the translated file(s) to the target repo, then commit via `git` on a branch (do not silently overwrite an existing pipeline file — that is an override).
3. Do **not** copy secret values referenced by the pipeline; list the secret **names** the pipeline needs so the user re-enters them (secret values are never migrated — see the fidelity note below).

## Fidelity limit (surface in the dry-run)

CI/CD secret *values* are never copied. GitHub and Azure DevOps never return secret values (write-only); only GitLab exposes CI/CD variable values. Report the secret **names/scoping** the pipeline references so the user re-enters values on the target. Never print a secret value.

## Cross-platform notes

- Marketplace actions / ADO tasks / GitLab templates rarely have 1:1 equivalents — translate the intent, leave a comment where a manual choice is needed.
- Self-hosted runners/agents and environments/service connections do not migrate; note them as manual setup.

## Verify

- Confirm the translated file exists at the correct path on the target and lint/validate it if the CLI supports it. Remind the user it still needs review and its secrets re-entered.
