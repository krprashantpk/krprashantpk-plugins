---
name: migrate-repos
description: 'Migrate a Git repository with its full history, branches, tags, and Git LFS objects across Azure DevOps, GitHub, and GitLab in any direction. Use when moving a repository, its branches, tags, or large files between these platforms (for example "migrate the repo to GitHub", "mirror branches and tags to GitLab", "bring the LFS objects over"). Self-contained: always runs a read-only dry-run first, migrates only what the user selected, confirms every destructive action, and never uses Personal Access Tokens. Not for pull requests, issues, branch policies, or pipelines.'
license: MIT
---

# Migrate `repo` — repositories, branches, tags, LFS

Move a Git repository and all its history, branches, tags, and large files between Azure DevOps, GitHub, and GitLab, in any direction. Git history preserves original commit authors/timestamps because it is moved with `git`, not recreated.

## Operating rules — never break these

1. **Dry-run, always.** Begin with a read-only preview that touches nothing on the target: enumerate the source, inspect the target, and write out exactly what would be created, changed, skipped, or cannot migrate. Proceed only on explicit user approval.
2. **Only what the user selected.** This skill migrates the repository (`repo`). Never expand into pull requests, issues, policies, or pipelines without asking.
3. **Confirm every destructive action.** Report each write before performing it and get explicit confirmation.
4. **Discover source and target first.** Verify the source exists and is migratable, and that the target has no existing data. If the target already has data, never auto-override — ask the user for explicit override confirmation.
5. **No Personal Access Tokens.** Verify the required CLI tools are installed and already signed in. If either side is not signed in, stop and ask the user to authenticate. Reuse an existing signed-in session and verify both source and target are authenticated before doing anything. **A CLI sign-in is not a git-transport sign-in** — a working `az`/`gh`/`glab` session does not guarantee `git push` can authenticate. Resolve git-transport auth explicitly before any clone or push. See [Git-transport authentication] below.

## Git-transport authentication

Being signed into a CLI (`az`, `gh`, `glab`) authenticates *API* writes — it does **not** authenticate the `git` transport used by clone/fetch/push. Resolve git-transport auth explicitly, and ask the user which path to take, before any clone or push.

- **CLI sign-in ≠ git-transport sign-in.** `az repos create`, `gh repo create`, or `glab` API calls can all succeed while `git push` still fails, because they use different credentials.
- **Azure DevOps — do not push with an `az` bearer token.** An AAD access token (e.g. `az account get-access-token --resource 499b84ac-1321-427f-aa17-267ca6975798`) can return **403 even against the REST API** on MSA / personal-tenant-backed orgs, because the AAD identity does not map to the org — even though `az repos ...` writes with the *same* session succeed. This mismatch is misleading; do not chase it.
- **Preferred ADO transport — ask the user first:**
  - **SSH** — use when the user already has an SSH key registered on the target platform. Clone and push over the SSH URL (`git@ssh.dev.azure.com:...`).
  - **Git Credential Manager** — otherwise, let GCM (`credential.helper=manager`) drive its interactive browser sign-in for the HTTPS remote. This is the reliable fallback.
- Confirm with the user which transport (SSH vs. Git Credential Manager) to use before running any clone or push, and verify `git ls-remote <url>` succeeds on **both** source and target before proceeding.

## Workflow

1. **Preflight.** Confirm the source and target coordinates (Azure DevOps `org/project/repo` · GitHub `owner/repo` · GitLab `group/project`), resolve git-transport auth (see [Git-transport authentication]), and confirm that both platforms are authenticated.
   - **Azure DevOps target needs org + project.** "Same name" only supplies the repo name. If the target is ADO and the org and/or project are unknown, **ask for them before the dry-run** — never guess.
   - **Azure DevOps projects ship a default same-named repo.** A new ADO project auto-creates an empty repo named after the project, so listing repos may show an existing same-named repo. Confirm with the user whether to migrate **into that existing repo** or **create a new repo** named after the source repo before writing anything.
2. **Discover & classify (read-only).** Enumerate branches, tags, LFS, and the default branch on the source, then inspect the target and **classify its state** — empty/new, shared-history-behind, or divergent/unrelated — to pick the matching reference document. See [Target state & reference routing] below.
3. **Dry-run report.** Present the **case-specific** dry run report: what would be created, skipped, or cannot migrate, plus conflicts and manual follow-ups. Nothing is written, so stop for explicit approval.
4. **Confirm & execute.** After approval, confirm each destructive write group, then perform it. On a conflict, offer **abort** or **override**; an override needs a second explicit confirmation naming exactly what will be replaced.
5. **Verify & summarize.** Re-read the target, compare it to the source, and list manual follow-ups.

## Unreachable or misconfigured source remote

`git clone --mirror <source-url>` assumes the source remote is reachable. If the clone/fetch fails (dead, moved, or misconfigured remote) but a **complete local clone already exists**, that local clone can serve as the migration source — but only after proving it is complete and in sync:

- **Non-shallow:** `git rev-parse --is-shallow-repository` must return `false` (a shallow clone is missing history and must not be used).
- **In sync with tracking refs:** for each branch, `git rev-parse <branch>` must equal `git rev-parse <branch>@{upstream}` (or the last-fetched `origin/<branch>`), proving no un-fetched commits are missing.

Only when **both** checks hold, migrate directly from the local clone (add the target as a remote and push from it). If either check fails, stop and ask the user — never migrate from a shallow or out-of-sync clone.

## Target state & reference routing

After discovering the source, inspect the target and classify it, then route to the matching reference. Add the target as a remote in the source mirror clone and fetch (this reads the target and writes only to the *local* clone — nothing is written to the target), then compare tips per branch:

```bash
git remote add target <target-url>
git fetch target                       # downloads target objects locally; writes nothing to the target
# for each branch <b> present on both sides:
git rev-list --count target/<b>..<b>   # +commits on source to ADD
git rev-list --count <b>..target/<b>   # commits ONLY on target (divergence)
git merge-base <b> target/<b>          # empty ⇒ unrelated histories
```

Classify each branch, then the target as a whole, and route:

| Target state | How to recognise it & what it means | Reference |
| --- | --- | --- |
| **Empty / new** | The target has no refs, so there is nothing to overwrite. A clean create — push everything with no risk to existing data. | [references/target-new-or-divergent.md](references/target-new-or-divergent.md) |
| **Shared history, behind** | Every branch is new, already up-to-date, or fast-forwardable — the source is ahead (`target/<b>..<b> > 0`) with no commits only on the target (`<b>..target/<b> == 0`), i.e. the target is an ancestor of the source. A non-destructive fast-forward add. | [references/target-shared-history.md](references/target-shared-history.md) |
| **Divergent / unrelated** | At least one branch has commits the source lacks (`<b>..target/<b> > 0`) or shares no merge-base. A fast-forward is impossible — the only options are **abort** (default) or a destructive **override** (which needs explicit user approval). | [references/target-new-or-divergent.md](references/target-new-or-divergent.md) |

The routed reference carries the **case-specific dry-run table, execute commands, and verification**. If a *behind* migration's non-force push is later rejected as non-fast-forward, that branch is actually divergent — abort and re-route to the new-or-divergent reference.

> **Divergent / unrelated history requires explicit user approval.** A force-replace (override) discards the target's existing commits and is effectively irreversible. Never override automatically: default to **abort**, and proceed only after the user explicitly approves an override, naming exactly which refs will be replaced.

