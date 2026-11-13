---
name: code-story-implementer
description: Use when the user wants to execute, implement, build, or close out a planned work item or backlog story. Reads an approved standalone plan under .backlog/technical-plan/ or a legacy story with an embedded Technical Implementation Plan, makes code/doc changes, runs tests, and appends an Implementation Summary and Closing Comment to the plan or legacy story. Triggers include "implement this story", "execute the plan", "do UC7", "ship this", and "close out the story". Requires an approved technical plan.
---

# CodeStory Implementer

Execute an approved technical plan end to end: make the code and documentation changes required by its architecture and acceptance outcomes, validate them, then record what was actually done.

The workflow is project-agnostic. Follow the target repository's language, frameworks, tooling, and conventions rather than imposing bundled implementation defaults.

## Overview

Given an approved plan under `.backlog/technical-plan/`, or a legacy story under `.backlog/` with an embedded *Technical Implementation Plan*, this skill:

1. Confirms the plan, approval, scope, and acceptance outcomes.
2. Detects repository conventions and translates the high-level approach into concrete changes.
3. Implements the approved architecture in delivery-sequence order, including tests and related documentation.
4. Runs applicable validation and checks the acceptance outcomes.
5. Records implementation status, verification, and follow-ups in `## Implementation Summary` and `## Closing Comment` in the standalone plan or legacy story.

It does *not* invent new acceptance criteria or expand scope beyond the plan. Out-of-scope discoveries are recorded in *What was missed / out of scope*, not silently implemented.

## Prerequisites

* An approved standalone plan exists under `.backlog/technical-plan/`, or a legacy story contains an approved embedded Technical Implementation Plan.
* The target repository is available for implementation and validation.
* The user has identified which plan or planned work item to implement.

No prior use of `code-story-author` is required. The source work item may be supplied text, a local file, or an issue/tracker reference.

## Required Steps

### Step 1: Load and Validate the Plan

1. Resolve the supplied plan path. When given a work-item reference, find its matching plan under `.backlog/technical-plan/` using the source reference and title; do not assume a title match alone identifies the same item. If the target is ambiguous, ask.
2. Read the full plan and any available source work item using read-only tools. Establish the agreed scope and acceptance outcomes from that context without requiring particular section headings. For a legacy story, include its Description and Acceptance Criteria. If an inaccessible source leaves material requirements unclear, ask the user to supply them rather than inventing details.
3. Confirm approval of the current plan from its approval record or the conversation. If approval is unclear or the design has changed since approval, ask before implementation. If the plan is missing or lacks material design decisions, stop and direct the user to `code-story-technical-plan`. A high-level plan does not require a file-by-file checklist or prescribed signatures.
4. Use the standalone plan as the closeout record, leaving its source work item unchanged. For a legacy embedded plan, use the story file as the closeout record.
5. If an Implementation Summary or Closing Comment already contains non-placeholder content, ask whether to update it in place or preserve it and record a follow-up pass.

### Step 2: Detect Repo Conventions and Tooling

1. Treat automatically loaded repository instructions as binding. Read available architecture, project-structure, README, and contributor documentation, including `ARCHITECTURE.md` or `architecture.md`, `PROJECT_STRUCTURE.md`, `README.md` or `readme.md`, `CONTRIBUTING.md`, and applicable `AGENTS.md` files when present.
2. Inspect relevant manifests, configuration, module boundaries, naming, tests, and tooling. Identify the repository's dependency manager, environment setup, test commands, linter, type checker, and formatter where applicable.
3. Consult relevant previous technical plans and their revision history when needed to understand decisions or compatibility constraints. Check that the approved approach still fits the current codebase; surface material conflicts before making changes.

### Step 3: Implement the Plan

1. Follow the approved architecture and delivery sequence, respecting dependencies and acceptance outcomes.
2. Trace the behavior's owning code, callers, dependencies, tests, configuration, and documentation as far as the work requires. Translate each high-level phase into concrete changes using repository conventions. Read current files before editing and preserve unrelated local changes.
3. Resolve routine implementation details within the approved design. If a material decision is ambiguous or requires changing the architecture, scope, or accepted assumptions, pause the affected work and obtain approval of the revised approach through `code-story-technical-plan` before implementing it.
4. Keep edits focused on the plan. Apply approved refactors, boundary changes, and folder reorganizations when required; do not refactor unrelated code or introduce unapproved abstractions.
5. Update tests, configuration, and related documentation for the implemented behavior, including changes required by repository instructions even when the plan only mentions production code.
6. When implementation exposes new, changed, or stale version-sensitive technical facts, verify them against current official documentation for the repository's declared versions. If a fact remains unverified, ask before relying on an explicit assumption. Do not silently substitute a different design or upgrade dependencies beyond the approved scope.

### Step 4: Validate

1. Run the checks required by the plan and repository, starting with the smallest relevant tests, lint, type checks, or build. Broaden validation when required by the change's impact or targeted results.
2. Verify each acceptance outcome and relevant architectural contracts, integrations, compatibility, and migration behavior. Distinguish verified outcomes from those that require an unavailable environment or external action.
3. Fix failures caused by or tightly coupled to the changes and re-run the affected checks. Do not fix unrelated pre-existing failures; report them as blockers when they prevent required validation.
4. Report failed, skipped, unavailable, or inconclusive checks explicitly. Do not claim completion while required checks or in-scope acceptance outcomes remain unverified.

### Step 5: Append Implementation Summary and Closing Comment

1. After implementation and validation attempts, edit the closeout record selected in Step 1. Append or update `## Implementation Summary` and `## Closing Comment` using the *Closeout Template* below and the user's choice for existing closeout content. Preserve the approved plan, its approval status, and source context.
2. Under `### What was added`, list concrete changes actually made: files added, modified, or removed, key behaviors, test coverage, and documentation updates. Reflect reality rather than intent.
3. Under `### What was missed / out of scope`, distinguish unfinished in-scope work from agreed deferrals and out-of-scope discoveries. Include external follow-ups and acceptance outcomes that could not be verified. State when none remain.
4. Record the implementation status as Complete, Incomplete, or Blocked. Use Complete only when all in-scope acceptance outcomes and required checks pass. Record exact validation commands, results, and any checks not run with their reasons; partial work must remain explicitly incomplete or blocked.
5. Write a 2-4 sentence Closing Comment the user can paste into a PR description or backlog tracker. State the actual user-visible outcome, implementation and verification status, and follow-ups without implying closure for incomplete or blocked work.
6. Confirm the closeout record was saved successfully. Report write failures explicitly; do not claim the record was updated if persistence failed.

### Step 6: Report Back

1. Link to the updated closeout record and list the files changed in the codebase.
2. State the implementation status and verification results, including blockers or unavailable checks.
3. Call out any open follow-ups recorded in *What was missed / out of scope*.

## Required Protocol

1. Implement only the approved approach, scope, and acceptance outcomes. Record out-of-scope discoveries rather than silently implementing them.
2. Never mark work complete or closed with failing required checks or unverified in-scope acceptance outcomes. Record partial or blocked results honestly.
3. Follow the target repository's instructions, language, frameworks, and tooling. Do not impose Python-specific or other cross-project defaults.
4. Modify only implementation-related files and the selected local closeout record. Preserve the approved plan content and do not modify an external tracker or the source work item of a standalone plan.
5. Ask about material design or scope ambiguity; do not treat ordinary implementation detail as a missing architectural plan.

## Closeout Template

````markdown
## Implementation Summary

Implementation status: <Complete | Incomplete | Blocked>

### What was added

* <Concrete change actually made, mirroring the plan but reflecting reality.>
* <...>

### What was missed / out of scope

* <Unfinished in-scope work, agreed deferrals, out-of-scope discoveries,
  external follow-ups, and unverified acceptance outcomes, or None.>

Verification: <exact commands and results; checks not run and reasons>.

## Closing Comment

<2-4 sentences: actual outcome, implementation and verification status,
and explicit follow-ups. Do not imply closure if incomplete or blocked.>
````

## Response Format

After completing the work, return:

* Link to the updated standalone plan or legacy story used as the closeout record.
* State whether implementation is complete, incomplete, or blocked.
* List code, test, configuration, and documentation files modified, added, or removed.
* Report validation commands and results, including checks not run and their reasons.
* List follow-ups from *What was missed / out of scope*.
