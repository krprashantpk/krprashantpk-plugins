# Code-Shift

> A GitHub Copilot CLI plugin that migrates project resources **between Azure DevOps, GitHub, and GitLab — in any direction**.

Moving a project between DevOps platforms is painful: the code is easy to push, but the pull requests, branch policies, pipelines, and issues around it usually get lost. Code-Shift treats a migration as a guided, conversational workflow. You say *from where*, *to where*, and *what*, and it translates each platform's model and moves the resources.

## Non-negotiable guardrails

Code-Shift is built around three rules it never breaks:

1. **Dry-run first, always.** Every migration begins with a read-only preview that touches nothing on the target. You approve before anything is written.
2. **Only the selected resources.** You choose which resource types to migrate (e.g. just `repo` and `prs`). A subset migration is fully supported, not a lesser path.
3. **Confirm every destructive action.** Each write to the target is reported and confirmed before it happens. Existing target data is never auto-overridden.

## Migratable resources

| Key | Resource | Notes |
| --- | --- | --- |
| `repo` | Repositories, branches, tags, LFS | Full history via `git` (preserves commit authorship) |
| `prs` | Pull / merge requests + comments | Recreated as the authenticated user; original author/date noted |
| `policies` | Branch protection / branch policies | Mapped across each platform's rule model |
| `pipes` | CI/CD pipeline definitions | YAML **translated** per platform (review required) |
| `issues` | Issues / work items + labels | Labels, state, and links preserved where possible |

**Out of scope (never migrated):** CI/CD secret *values*, wikis, build/release artifacts, packages. Secret *names/scoping* are reported so you can re-enter values manually.

## Authentication — no PATs

Code-Shift reuses an already-authenticated session per platform. Sign in first with `az login`, `gh auth login`, and/or `glab auth login`. It never asks for, stores, or generates a Personal Access Token.

## Usage

Install from the marketplace this repo publishes, then describe your migration in natural language:

```sh
copilot plugin install code-shift@krprashantpk-plugins
```

- **Skills** — describe the move in natural language; the skills auto-trigger. Each resource type has its own **self-contained** skill (full guardrails, workflow, and platform commands): `migrate-repos`, `migrate-pull-requests`, `migrate-branch-policies`, `migrate-pipelines`, `migrate-issues`. Example: *"Migrate my Azure DevOps repo and PRs to GitHub."*

## Project structure

```
code-shift/
├── plugin.json                              # Plugin manifest (name, version, skills path)
├── AGENTS.md                                # Primary instructions: identity, guardrails, work routing
├── architecture.md                          # Component + workflow diagrams
├── README.md                                # This file
└── skills/                                   # One self-contained skill per resource type
    ├── migrate-repos/                        # repo skill (git mirror, branches, tags, LFS)
    │   ├── SKILL.md                          # workflow + target-state classification & routing
    │   └── references/                       # case-specific dry-run + execute + verify guides
    │       ├── target-new-or-divergent.md    # new target, or divergent history (create · abort · override)
    │       └── target-shared-history.md      # target behind on shared history (fast-forward add)
    ├── migrate-pull-requests/SKILL.md       # prs skill (PR/MR + comments)
    ├── migrate-branch-policies/SKILL.md     # policies skill (rule-model mapping)
    ├── migrate-pipelines/SKILL.md           # pipes skill (YAML translation, ADO Classic)
    └── migrate-issues/                       # issues skill (issues/work items + labels)
        ├── SKILL.md                          # workflow: process/type/state mapping, hierarchy, attribution
        └── references/
            └── github-to-ado-script-template.md  # GH→ADO bulk script (CLI escaping, sub-issues, resumable map)
```

## Tooling principle

Default to the platform **CLI (with `*/api` REST passthrough)** for both reads and writes. Only native CLIs and REST APIs are used — no vendor "import into" engines, which are one-directional and mostly PAT-based.

## License

[MIT](../../LICENSE)
