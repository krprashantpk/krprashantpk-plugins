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
5. **No Personal Access Tokens.** Verify the required CLI tools are installed and already signed in. If either side is not signed in, stop and ask the user to authenticate. Reuse an existing signed-in session and verify both source and target are authenticated before doing anything.

## Workflow

1. **Preflight.** Confirm the source and target coordinates (Azure DevOps `org/project/repo` · GitHub `owner/repo` · GitLab `group/project`), and confirm that both platforms are authenticated.
2. **Discover & classify (read-only).** Enumerate branches, tags, LFS, and the default branch on the source, then inspect the target and **classify its state** — empty/new, shared-history-behind, or divergent/unrelated — to pick the matching reference document. See [Target state & reference routing] below.
3. **Dry-run report.** Present the **case-specific** dry run report: what would be created, skipped, or cannot migrate, plus conflicts and manual follow-ups. Nothing is written, so stop for explicit approval.
4. **Confirm & execute.** After approval, confirm each destructive write group, then perform it. On a conflict, offer **abort** or **override**; an override needs a second explicit confirmation naming exactly what will be replaced.
5. **Verify & summarize.** Re-read the target, compare it to the source, and list manual follow-ups.

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

