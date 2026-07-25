---
name: migrate-branch-policies
description: 'Migrate branch protection rules / branch policies across Azure DevOps, GitHub, and GitLab in any direction by mapping each platform''s rule model. Use when moving branch protection, required reviewers/approvals, required status checks, linear-history rules, or push restrictions between these platforms (for example "carry over the branch policies", "protect main on GitHub like it is in Azure DevOps"). Self-contained: always runs a read-only dry-run first, migrates only what the user selected, confirms every destructive action, and never uses Personal Access Tokens. Not for repositories, pull requests, issues, or pipelines.'
license: MIT
---

# Migrate `policies` — branch protection / branch policies

Map each platform's branch-rule model onto the target across Azure DevOps, GitHub, and GitLab, in any direction. Rule models differ, so this is a **translation**, not a copy; some rule types have no equivalent and must be flagged. Apply policies after the branches exist on the target — migrate the repository first.

## Operating rules — never break these

1. **Dry-run, always.** Begin with a read-only preview that touches nothing on the target: read the source branch rules, inspect the target, and write out exactly what would be created, changed, skipped, or cannot migrate. Proceed only on explicit user approval.
2. **Only what the user selected.** This skill migrates branch policies (`policies`). Never expand into repositories, pull requests, issues, or pipelines without asking.
3. **Confirm every destructive action.** Report each write before performing it and get explicit confirmation.
4. **Discover source and target first.** Verify the source branch rules exist and are migratable, that the protected branches already exist on the target, and inspect the target's existing protection. If the target branch already has protection, never auto-override — ask the user for explicit override confirmation.
5. **No Personal Access Tokens.** Verify the required CLI tools (`az`, `gh`, `glab`) are installed and already signed in. If either side is not signed in, stop and ask the user to authenticate. Reuse an existing signed-in session and verify both source and target are authenticated before doing anything.

## Workflow

1. **Preflight.** Confirm the source and target coordinates (Azure DevOps `org/project/repo` · GitHub `owner/repo` · GitLab `group/project`), confirm both platforms are authenticated, and confirm the protected branches already exist on the target (migrate the repository first if not).
2. **Discover (read-only).** Read the source branch rules, then inspect the target's existing protection so nothing is silently overwritten.
3. **Dry-run report.** Present each rule mapped to the target model, what cannot map, and conflicts. Nothing is written, so stop for explicit approval.
4. **Confirm & execute.** After approval, confirm each destructive write group, then apply it. On existing target protection, offer **abort** or **override**; an override needs a second explicit confirmation naming exactly what will be replaced.
5. **Verify & summarize.** Re-read the target protection, compare it to the source, and list anything downgraded or dropped.

Prefer the platform **CLI (with `*/api` REST passthrough) for writes** and the **MCP server for read/preview** — `code-shift-github`, `code-shift-azure-devops`, `code-shift-gitlab` (see `mcp.json`).

## Model mapping (approximate)

| Concept | Azure DevOps (branch policy) | GitHub (branch protection) | GitLab (protected branch + push rules) |
| --- | --- | --- | --- |
| Require PR before merge | "Require a minimum number of reviewers" | "Require a pull request before merging" | Merge access levels |
| Required approvals count | Minimum reviewers = N | Required approving reviews = N | Approvals required = N |
| Required status checks | Build validation policy | Required status checks | Pipeline must succeed |
| Linear history / no force-push | "Limit merge types" / squash | "Require linear history" | "Do not allow force push" |
| Restrict who can push | Security/permissions | Restrict who can push | Allowed to push access level |
| Comment resolution | "Check for comment resolution" | Conversation resolution | (no direct equivalent) |

Anything without a target equivalent → list under **Cannot migrate** in the dry-run.

## Discover (read-only)

- Azure DevOps: `az repos policy list --repository-id <id>`; MCP repositories (partial).
- GitHub: `gh api repos/{o}/{r}/branches/{branch}/protection`.
- GitLab: `glab api projects/:id/protected_branches`.

## Dry-run entries

- Will create: N rules per protected branch, mapped to the target model (show the mapping).
- Cannot migrate: rule types with no target equivalent (list them).
- Conflicts: existing protection on the target branch → abort or override.

## Execute (confirm each write — rule 3)

Branch policy writes are frequently via REST passthrough — confirm before applying:

- GitHub: `gh api -X PUT repos/{o}/{r}/branches/{branch}/protection --input rule.json`.
- GitLab: `glab api -X POST projects/:id/protected_branches -f name=<branch> -f push_access_level=<n> -f merge_access_level=<n>`.
- Azure DevOps: `az repos policy <type> create ...` (e.g. `approver-count`, `build`) or `az devops invoke` for policy types without a first-class command.

## Cross-platform notes

- Apply policies **after** branches exist on the target (so protected branches resolve).
- Overriding existing target protection is an **override** — require the second explicit confirmation.
- Team/identity-based restrictions rarely map across platforms (different identity systems); flag these for manual setup.

## Verify

- Re-read the target branch protection and confirm each mapped rule is present; report anything downgraded or dropped.
