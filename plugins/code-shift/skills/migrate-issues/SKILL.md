---
name: migrate-issues
description: 'Migrate issues (GitHub / GitLab) or work items (Azure DevOps) with their labels, state, and links across platforms in any direction. Use when moving issues, work items, or labels between Azure DevOps, GitHub, and GitLab (for example "port the GitHub issues to GitLab", "recreate the work items on GitHub", "migrate the labels"). Self-contained: always runs a read-only dry-run first, migrates only what the user selected, confirms every destructive action, and never uses Personal Access Tokens. Not for repositories, pull requests, branch policies, or pipelines.'
license: MIT
---

# Migrate `issues` — issues / work items + labels

Recreate issues (GitHub/GitLab) or work items (Azure DevOps) with their labels, state, and links, across Azure DevOps, GitHub, and GitLab in any direction. Recreate labels first so issues can reference them.

## Operating rules — never break these

1. **Dry-run, always.** Begin with a read-only preview that touches nothing on the target: enumerate the source issues/labels, inspect the target, and write out exactly what would be created, changed, skipped, or cannot migrate. Proceed only on explicit user approval.
2. **Only what the user selected.** This skill migrates issues/work items and labels (`issues`). Never expand into repositories, pull requests, policies, or pipelines without asking.
3. **Confirm every destructive action.** Report each write before performing it and get explicit confirmation.
4. **Discover source and target first.** Verify the source issues and labels exist and are migratable, and inspect the target for already-present labels and issues. If a matching label or issue already exists, never auto-override — ask the user for explicit override confirmation.
5. **No Personal Access Tokens.** Verify the required CLI tools (`az`, `gh`, `glab`) are installed and already signed in. If either side is not signed in, stop and ask the user to authenticate. Reuse an existing signed-in session and verify both source and target are authenticated before doing anything.

## Workflow

1. **Preflight.** Confirm the source and target coordinates (Azure DevOps `org/project/repo` · GitHub `owner/repo` · GitLab `group/project`) and confirm both platforms are authenticated.
2. **Discover (read-only).** Enumerate labels and issues/work items on the source, then inspect the target for already-present labels/issues so nothing is duplicated. Recreate labels before issues.
3. **Dry-run report.** Present what would be created, skipped, or cannot migrate faithfully, plus conflicts. Nothing is written, so stop for explicit approval.
4. **Confirm & execute.** After approval, confirm each destructive write group (labels, then issues), then perform it. On a conflict, offer **abort** or **override**; an override needs a second explicit confirmation naming exactly what will be replaced.
5. **Verify & summarize.** Re-read the target, compare counts to the source, and list manual follow-ups.

Prefer the platform **CLI (with `*/api` REST passthrough) for writes** and the **MCP server for read/preview** — `code-shift-github`, `code-shift-azure-devops`, `code-shift-gitlab` (see `mcp.json`).

## Fidelity limit (surface in the dry-run)

Recreated issues/work items post as the **authenticated user** with new timestamps. Prefix each with the original author and date, e.g. `> Originally created by @alice on 2025-03-02`. Cross-item links (`#123`) may need re-mapping to the new target ids.

## Model mapping

| Concept | Azure DevOps | GitHub | GitLab |
| --- | --- | --- | --- |
| Item | Work item (Bug/Task/User Story…) | Issue | Issue |
| Category | Work item type | Label | Label |
| State | State (New/Active/Closed) | Open/Closed (+ state reason) | Open/Closed |
| Grouping | Area/Iteration path | Milestone/Project | Milestone/Epic |

Work item *types* have no direct GitHub/GitLab equivalent — map them to labels (e.g. `type:bug`) and note it.

## Discover (read-only)

- Labels: `az boards` (tags) · `gh label list` · `glab label list`.
- Items: `az boards query --wiql ...` / `az boards work-item show` · `gh issue list --state all --json ...` · `glab issue list`.
- Target: which issues/labels already exist (match by title)? Record conflicts to avoid duplicates.

## Dry-run entries

- Will create: N labels, M issues/work items (with mapped types/states).
- Will skip: already-present labels/issues (idempotency).
- Cannot migrate faithfully: authorship/timestamps, ADO work-item-type semantics, cross-links.

## Execute (confirm each write — rule 3)

1. **Labels first** (confirm the batch): `gh label create` · `glab label create` · ADO tags are created implicitly on work items.
2. **Issues/work items** (confirm the batch):
   - GitHub: `gh issue create --title "<t>" --body "<body + author note>" --label <l>`.
   - GitLab: `glab issue create --title "<t>" --description "<body + author note>" --label <l>`.
   - Azure DevOps: `az boards work-item create --type <T> --title "<t>" --description "<body + author note>"`.
3. Set closed state where the source item is closed (`gh issue close --reason ...`, `glab issue close`, ADO state update). Always set a state reason when closing.

## Cross-platform notes

- Preserve label color/description where the target supports it.
- Re-map issue references (`#123`) to the new target ids where possible; otherwise note the original id in the body.

## Verify

- Compare target issue/label counts to the source; confirm states and labels applied. Report any items whose type/links were downgraded.
