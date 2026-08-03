---
name: backlog-story-technical-plan
description: Use when the user wants to add the technical breakdown to an existing backlog story, plan an implementation, identify affected files, or design a file-by-file change set. Reads a story under .backlog/ with Description and Acceptance Criteria, verifies version-sensitive decisions against current official documentation, and appends a confirmed Technical Implementation Plan. Triggers include "plan this story", "break it down", "what files change", and "design the implementation". Requires background authored by backlog-story-author.
---

# Backlog Story Technical Plan

Append a concrete, file-by-file *Technical Implementation Plan* to an existing backlog story. This skill is project-agnostic; it adapts to the language, frameworks, and conventions of the current repository.

## Overview

Given a story file under `.backlog/` that already has *Description* and *Acceptance Criteria*, this skill:

1. Loads the story and confirms the background is sufficient to plan against.
2. Explores the codebase to identify every file the change will touch.
3. Optionally runs a focused clarifying-question round on technical-only ambiguities.
4. Appends a `## Technical Implementation Plan` section to the story, grouped by `### Modified files`, `### New files`, and `### Removed files`.

It does *not* edit production code. It does *not* rewrite the Description or Acceptance Criteria — if those need changes, send the user back to the `backlog-story-author` skill.

## Prerequisites

* A story file exists under `.backlog/` with at minimum `## Description` and `## Acceptance Criteria` populated.
* The user has identified which story to plan (by path or id). If multiple unplanned stories exist, ask the user which one.

## Required Steps

### Step 1: Load the Story

1. Resolve the target story file under `.backlog/`. If unspecified, list candidates and ask.
2. Read the full story. Confirm the *Description* and *Acceptance Criteria* are present and non-empty. If either is missing or vague, stop and tell the user to run `backlog-story-author` first.
3. If the story already contains a `## Technical Implementation Plan` section, ask whether to replace it or refine in place.

### Step 2: Detect Repo Conventions

1. Read `README.md`, `architecture.md`, and `CONTRIBUTING.md` when present, and treat automatically loaded repository instructions as binding.
2. Inspect the repository layout, package manifests, configuration, module boundaries, naming, tests, and tooling to identify established patterns.
3. Match those patterns in the plan. Propose missing or updated documentation only when the story or repository instructions require it.

### Step 3: Explore the Codebase

1. Start from the code that owns the requested behavior, then trace its callers, dependencies, tests, configuration, infrastructure, and documentation only as far as the story requires.
2. Record exact paths, identifiers, signatures, call sites, contracts, and related tests. Avoid generic directions such as "update the relevant service".
3. Classify every affected file as modified, new, or removed.

### Step 4: Verify Current Technical Facts Online (Mandatory When Applicable)

1. Identify every recommendation that depends on changeable external facts, including APIs, SDKs, libraries, cloud services, security guidance, configuration syntax, compatibility, support status, deprecations, or migration behavior.
2. Research those facts online using the latest applicable official documentation, release notes, support matrices, or specifications. Do not rely on model memory, blogs, or community examples as authority.
3. Prefer the latest stable option compatible with the repository's declared versions and constraints, not simply the newest available version. Retain the relevant product/library version, document title or URL, and verification date for the chat summary, not the story file.
4. If current official documentation cannot be accessed or does not resolve the fact, label it unverified and ask the user whether to proceed with an explicit assumption. Do not present it as confirmed.

### Step 5: Resolve Technical Ambiguities

1. Ask only material technical questions that the story, repository, and official documentation cannot answer.
2. Provide a recommended default with its trade-off instead of presenting an open-ended choice when evidence supports one.
3. Do not revisit business scope. If the user declines further questions, record unresolved assumptions in the plan.

### Step 6: Present the High-Level Plan and Confirm

1. Before editing the story, present the proposed design from an architecture perspective: affected components and their responsibilities, boundary changes, interactions and data/control flow, integration points, and how the approach fits the existing architecture.
2. Explain the key design decisions, alternatives considered, material trade-offs, risks, constraints, assumptions, and deferred work. Use a compact Mermaid diagram when component relationships or flow would otherwise be unclear.
3. After the architecture proposal, include the grouped modified/new/removed file list as the expected implementation footprint. The file list supports the design; it is not a substitute for the high-level plan.
4. When Step 4 applies, include the official sources, relevant versions, and facts they verify.
5. Ask the user to confirm, refine, or reject the plan. Incorporate requested changes and re-present it until explicitly confirmed; stop without modifying the story if rejected.

### Step 7: Append the Technical Implementation Plan

1. Edit the story file in place. Insert `## Technical Implementation Plan` immediately after `## Acceptance Criteria` and before any `## Implementation Summary` section if one already exists.
2. Expand the confirmed high-level plan using the template. Omit empty file groups.
3. For each file, specify exact identifiers, signatures, behavior, validation, logging, tests, configuration, and documentation changes as applicable. Every instruction must be executable without rediscovering the design.
4. Preserve material trade-offs and unresolved assumptions in the framing paragraph. Write only the resulting technical decisions and constraints to the story; do not add research citations or verification metadata.

### Step 8: Report Back

1. Print the workspace-relative path to the updated story file.
2. Summarise the plan in 3-6 bullets: scope, key files, notable trade-offs.
3. Point the user at `backlog-story-implementer` for execution.

## Required Protocol

1. Never modify production code; only edit the target story file under `.backlog/`.
2. Never edit `## Description` or `## Acceptance Criteria` in this skill — defer to `backlog-story-author`.
3. Never use model memory as evidence for a version-sensitive technical fact; verify it against current official online documentation as required by Step 4.
4. Never write the `## Technical Implementation Plan` until the user explicitly confirms the high-level plan presented in Step 6.
5. A grouped file list alone is not a high-level plan; present the architecture and design rationale before the implementation footprint.
6. Keep the written plan concrete, file-by-file, and consistent with repository conventions.

## Planning Principles

1. Prefer the smallest coherent change that satisfies the Acceptance Criteria; avoid speculative abstractions and unrelated refactors.
2. Reuse existing code and preserve established boundaries before proposing new helpers, types, or dependencies.
3. Keep dependencies explicit and add an abstraction only when it clarifies a real contract or supports existing multiple implementations.
4. Keep tests proportional to behavior and risk.

## Plan Template

````markdown
## Technical Implementation Plan

<One-paragraph framing of the approach, including any unresolved trade-offs.>

### Modified files

- `path/to/file.ext`
    - <Concrete change: signature, behaviour, logging, etc.>

### New files

- `path/to/new_file.ext` *(new)*
    - <Purpose and shape.>

### Removed files

- `path/to/old_file.ext`
    - <Reason for removal.>
````

## Response Format

After updating the file, return:

* **Story path** — workspace-relative path to the updated file.
* **Plan summary** — 3-6 bullets covering scope, key files, notable assumptions.
* **Verified sources** — official documentation used for version-sensitive decisions, or state that none was required.
* **Next step** — point the user at `backlog-story-implementer` to execute the plan.
