---
name: migrate-pull-requests
description: 'Migrate open pull requests (GitHub / Azure DevOps) or merge requests (GitLab) with their comment threads across platforms in any direction. Use when moving PRs/MRs between Azure DevOps, GitHub, and GitLab (for example "bring the open PRs over", "migrate merge requests to GitHub", "recreate the pull requests on GitLab"). Self-contained: always runs a read-only dry-run first, migrates only what the user selected, confirms every destructive action, and never uses Personal Access Tokens. Requires the branches to exist on the target first. Not for repositories, issues, branch policies, or pipelines.'
license: MIT
---

# Migrate `prs` — pull / merge requests + comments

Recreate open pull requests (GitHub/Azure DevOps) or merge requests (GitLab), including their comment threads, between Azure DevOps, GitHub, and GitLab in any direction. The PR/MR branches must already exist on the target — migrate the repository and its branches first.

## Operating rules — never break these

1. **Dry-run, always.** Begin with a read-only preview that touches nothing on the target: enumerate the source PRs/MRs and their comment threads, inspect the target, and write out exactly what would be created, changed, skipped, or cannot migrate. Proceed only on explicit user approval.
2. **Only what the user selected.** This skill migrates pull/merge requests (`prs`). Never expand into repositories, issues, policies, or pipelines without asking.
3. **Confirm every destructive action.** Report each write before performing it and get explicit confirmation.
4. **Discover source and target first.** Verify the source PRs/MRs exist and are migratable, that their branches already exist on the target, and that the matching PR/MR is not already present on the target. If a PR/MR already exists, never auto-override — ask the user for explicit override confirmation.
5. **No Personal Access Tokens.** Verify the required CLI tools (`az`, `gh`, `glab`) are installed and already signed in. If either side is not signed in, stop and ask the user to authenticate. Reuse an existing signed-in session and verify both source and target are authenticated before doing anything.

## Workflow

1. **Preflight.** Confirm the source and target coordinates (Azure DevOps `org/project/repo` · GitHub `owner/repo` · GitLab `group/project`), confirm both platforms are authenticated, and confirm the PR/MR branches already exist on the target (migrate the repository first if not).
   - **Azure DevOps target needs org + project.** "Same name" only supplies the repo name. If the target is ADO and the org and/or project are unknown, **ask for them before the dry-run** — never guess.
2. **Discover (read-only).** Enumerate open PRs/MRs and their comment threads on the source, then inspect the target for already-present PRs/MRs (match by source branch + title) so nothing is duplicated.
3. **Dry-run report.** Present what would be created, skipped, or cannot migrate, plus conflicts and manual follow-ups. Nothing is written, so stop for explicit approval.
4. **Confirm & execute.** After approval, confirm each destructive write group, then perform it. On a conflict, offer **abort** or **override**; an override needs a second explicit confirmation naming exactly what will be replaced.
5. **Verify & summarize.** Re-read the target, compare it to the source, and list manual follow-ups.

Prefer the platform **CLI (with `*/api` REST passthrough)** for both reads and writes.

## Fidelity limit (surface in the dry-run)

Recreated PRs/MRs and comments post as the **authenticated user**, not the original author. Prefix each with the original author and date, e.g. `> Originally opened by @alice on 2025-03-02`. Review state (approvals) generally cannot be faithfully reproduced. Git commit authorship inside the branches is unaffected (it moves with `git`).

## Discover (read-only)

- Source open PRs/MRs with source/target branches, title, body, and comments:
  - Azure DevOps: `az repos pr list --status active`, `az repos pr show --id <id>`.
  - GitHub: `gh pr list --state open --json ...`, `gh pr view <n> --json comments`.
  - GitLab: `glab mr list`, `glab mr view <iid>`.
- Target: which PRs/MRs already exist (match by source branch + title)? Record conflicts to avoid duplicates.

## Dry-run entries

- Will create: N PRs/MRs (source ids) + M comments.
- Will skip: already-present PRs/MRs (idempotency), and merged/closed ones unless the user asks to include them.
- Cannot migrate: authorship/timestamps (recreated as the authenticated user), approval/review state.

## Execute (confirm each write — rule 3)

Confirm before creating each PR/MR (or a clearly described batch):

- GitHub: `gh pr create --base <target> --head <source> --title "<t>" --body "<body + author note>"`.
- GitLab: `glab mr create --source-branch <s> --target-branch <t> --title "<t>" --description "<body + author note>"`.
- Azure DevOps: `az repos pr create --source-branch <s> --target-branch <t> --title "<t>" --description "<body + author note>"`.
- Then add comments in thread order via `gh pr comment` / `glab mr note` / `az repos pr comment` (or the respective `*/api`), each prefixed with the original author/date.

## Cross-platform notes

- Terminology maps directly: GitHub/ADO **pull request** ⇄ GitLab **merge request**.
- Only recreate PRs/MRs whose branches exist on the target; if a branch is missing, tell the user rather than silently creating it.
- Carry over draft state and labels where the target supports them (`--draft`; labels are managed by the `migrate-issues` skill).

## Verify

- List target PRs/MRs and compare against the source set; confirm comment counts per PR/MR.
