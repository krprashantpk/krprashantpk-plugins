---
name: backlog-story-implementer
description: Use when the user wants to execute, implement, build, or close out a planned backlog story. Reads a story under .backlog/ that already has Description, Acceptance Criteria, and a Technical Implementation Plan, then makes the actual code/doc changes, runs tests, and appends an "## Implementation Summary" and "## Closing Comment" to the story. Triggers include "implement this story", "execute the plan", "do UC7", "ship this", "close out the story". Requires the story to already have a Technical Implementation Plan authored by the backlog-story-technical-plan skill.
---

# Backlog Story Implementer

Execute a fully-planned backlog story end to end: make the code and documentation changes described in the story's Technical Implementation Plan, validate them, then record what was actually done.

## Overview

Given a story file under `.backlog/` that already has *Description*, *Acceptance Criteria*, and *Technical Implementation Plan*, this skill:

1. Loads and parses the plan.
2. Applies each change in the plan, file by file.
3. Runs the project's tests / linters / type checks.
4. Updates documentation files the plan calls out (for example `README.md`, `architecture.md`).
5. Appends `## Implementation Summary` (with `### What was added` and `### What was missed / out of scope`) and `## Closing Comment` to the story.

It does *not* invent new acceptance criteria or expand scope beyond the plan. Out-of-scope discoveries are recorded in *What was missed / out of scope*, not silently implemented.

## Prerequisites

* A story file exists under `.backlog/` with `## Description`, `## Acceptance Criteria`, and `## Technical Implementation Plan` all populated.
* The repository's tests can be run locally (or the user has indicated which command to use).
* The user has identified which story to implement.

## Required Steps

### Step 0: Load Implementation Defaults (Mandatory)

Before doing anything else, read `references/python-and-project-conventions.md`. Treat all repository instructions automatically supplied by Copilot as binding; if they contradict a step below or the bundled reference, the repository instructions win. Apply Python-specific defaults only to Python projects and files.

### Step 1: Load and Validate the Story

1. Resolve the target story file under `.backlog/`. If unspecified, list candidates that have a Technical Implementation Plan and ask.
2. Read the full story. Confirm Description, Acceptance Criteria, and Technical Implementation Plan are present and non-empty.
3. If the plan is missing or thin, stop and tell the user to run `backlog-story-technical-plan` first.
4. If `## Implementation Summary` already exists with non-placeholder content, ask whether to update in place or treat this as a follow-up pass.

### Step 2: Detect Repo Conventions and Tooling

1. Read root-level project files (`README.md`, `architecture.md`, `CONTRIBUTING.md`, and `AGENTS.md`) and use the automatically loaded repository instructions to learn local conventions and tooling.
2. Identify the test command, linter, type checker, formatter, and dependency manager. Use them rather than imposing patterns from other projects.
3. Resolve the applicable defaults from `references/python-and-project-conventions.md`, including environment management, logging, design, numeric precision, documentation synchronization, and Python docstring/type rules. Do not apply a default that conflicts with the target repository.

### Step 3: Implement the Plan

1. Walk the Technical Implementation Plan in order: `### Modified files`, then `### New files`, then `### Removed files`.
2. For each entry, make the exact change described. Read the current file before editing.
3. If a bullet is ambiguous or under-specified, stop and ask the user one focused question rather than guessing. Do not silently expand scope.
4. Keep edits minimal and aligned with the bullet — do not refactor unrelated code, do not add docstrings/comments to code you did not change, do not introduce new abstractions not called for by the plan.
5. Update tests and docs as the plan specifies. Add tests or documentation required by the target repository or bundled conventions even when the plan only mentions production code.

### Step 4: Validate

1. Run the project's tests, linter, and type checks. Report results.
2. If anything fails, fix the failure (sticking to the plan's intent) and re-run. Do not mark the story complete with failing checks.
3. Sanity-check the change against each Acceptance Criterion. Note any criterion that cannot be verified yet (for example needs a deployed environment) so it can be recorded.

### Step 5: Append Implementation Summary and Closing Comment

1. Edit the story file in place. Append (or replace placeholder content for) `## Implementation Summary` and `## Closing Comment` using the *Closeout Template* below.
2. `### What was added` — bullet every concrete change actually made: files added/modified/removed, key behaviours, test coverage, doc updates. Mirror the level of detail in the plan but reflect *reality*, not intent.
3. `### What was missed / out of scope` — bullet anything the plan called for that was not done, anything discovered mid-flight that was deferred, follow-ups for CI/infra/env config the repo does not own, and any acceptance criteria that could not be fully verified.
4. End with a verification line stating which tests/lints were run and the result (for example `Verification: full test suite — 128/128 passing`).
5. `## Closing Comment` — 2-4 sentence summary the user can paste into a PR description or backlog tracker. State the user-visible outcome, the verification status, and any explicit follow-ups.

### Step 6: Report Back

1. Print the workspace-relative path to the updated story file and a list of files changed in the codebase.
2. Surface verification results.
3. Call out any open follow-ups recorded in *What was missed / out of scope*.

## Required Protocol

1. Do not expand scope beyond the Technical Implementation Plan. Out-of-scope items go into *What was missed / out of scope*.
2. Never mark a story closed with failing tests, lints, or type checks.
3. Respect repository conventions discovered in Step 2.
4. Edit the story file under `.backlog/` only after the code changes succeed and validation is complete.
5. When in doubt about a plan bullet, ask one focused question — do not guess.

## Closeout Template

````markdown
## Implementation Summary

### What was added

- <Concrete change actually made, mirroring the plan but reflecting reality.>
- <...>

### What was missed / out of scope

- <Plan items deferred, mid-flight discoveries, env/CI/infra follow-ups, AC that could not be verified.>

Verification: <command(s) run> — <result>.

## Closing Comment

<2-4 sentences: user-visible outcome, verification status, explicit follow-ups.>
````

## Response Format

After completing the work, return:

* **Story path** — workspace-relative path to the updated story file.
* **Files changed** — list of code/doc files modified, added, or removed.
* **Verification** — which checks were run and the result.
* **Follow-ups** — anything left in *What was missed / out of scope*.
