---
name: code-story-technical-plan
description: Use when the user wants a high-level technical implementation plan for any supplied work item, story, issue, bug, feature, or task. Collaboratively develops an architecture-led approach, verifies version-sensitive decisions against current official documentation, and incorporates user feedback until explicitly approved. Only then creates .backlog/technical-plan/<story-title-slug>.md.
---

# CodeStory Technical Plan

Collaboratively develop a high-level technical plan for any supplied work item. Think from an architect's perspective, discuss the approach with the user, and document only the approved plan in `.backlog/technical-plan/<story-title-slug>.md`.

## Overview

Given a work item supplied as text, a local file, or an issue/tracker reference, this skill:

1. Confirms the work item's intent, scope, constraints, and acceptance outcomes with the user.
2. Explores the repository to understand the current architecture and relevant behavior, reads available architecture, project-structure, and README documentation, and reviews relevant previous technical plans and their history when available.
3. When needed, researches official documentation online for relevant patterns, services, and technologies, including verification of version-sensitive facts.
4. Proposes a high-level approach, including justified changes to existing code, component boundaries, or folder structure.
5. Incorporates feedback until the user explicitly approves the plan.
6. Creates a standalone technical-plan work item under `.backlog/technical-plan/`.

It does not implement code, change the supplied work item, or create or update items in external trackers. The plan is project-agnostic and follows the current repository's language, frameworks, and conventions.

## Prerequisites

* The user has supplied or identified a work item to plan. No particular location, section format, or authoring skill is required.
* The repository is available for architectural investigation.

## Required Steps

### Step 1: Understand the Supplied Work Item

1. Read the supplied text or file, or retrieve the identified issue using available read-only tools. If the reference cannot be accessed, ask the user to paste its contents; do not invent missing details.
2. Establish the title, problem, desired behavior, acceptance outcomes, constraints, and out-of-scope work. Clarify missing information directly with the user.
3. If an existing technical plan is supplied or discovered, ask whether to refine or replace it. Keep it unchanged until the revised plan is approved.

### Step 2: Detect Repo Conventions

1. Read available project documentation, including  `ARCHITECTURE.md`, `PROJECT_STRUCTURE.md`, and `README.md` (or `readme.md`) when present, and treat automatically loaded repository instructions as binding.
2. Review relevant previous technical plans and their available revision history, including standalone plans under `.backlog/technical-plan/` and legacy embedded plans under `.backlog/`. Extract useful decisions, rationale, constraints, and lessons, and check that they still match the current codebase.
3. Inspect the repository layout, package manifests, configuration, module boundaries, naming, tests, and tooling to identify established patterns.
4. Use those patterns as a starting point, not an absolute architectural constraint. Identify where the work item warrants changes to existing code, component boundaries, or folder structure.

### Step 3: Explore the Codebase

1. Start from the code that owns the requested behavior, then trace callers, dependencies, tests, configuration, infrastructure, and documentation only as far as the work item requires.
2. Identify component responsibilities, contracts, integration points, and data/control flow. Ground the design in actual modules and paths where useful without producing an exhaustive file inventory or prescribing signatures.
3. Evaluate whether extending the current design or restructuring it better serves the work item. Explain the benefit, migration impact, and compatibility implications of any proposed reorganization.

### Step 4: Research Current Technical Guidance When Needed

Use online research when design choices need external evidence or recommendations depend on version-sensitive facts; otherwise skip this step.

1. Identify open design questions about patterns, services, and technologies, and every recommendation that depends on changeable external facts, including APIs, SDKs, libraries, cloud services, security guidance, configuration syntax, compatibility, support status, deprecations, or migration behavior.
2. Research those questions and facts online using the latest applicable official documentation, release notes, support matrices, or specifications. Use the findings to evaluate relevant alternatives and trade-offs. Do not rely on model memory, blogs, or community examples as authority.
3. Prefer the latest stable option compatible with the repository's declared versions and constraints, not simply the newest available version. Retain the relevant product/library version, document title or URL, and verification date for the chat summary.
4. If current official documentation cannot be accessed or does not resolve the fact, label it unverified and ask the user whether to proceed with an explicit assumption. Do not present it as confirmed.

### Step 5: Develop the Approach Interactively

1. Discuss material design decisions and ambiguities that the work item, repository, and official documentation cannot resolve. Use the available question tool for user input.
2. Recommend an approach with its rationale and trade-offs. Explore alternatives when they materially affect complexity, maintainability, compatibility, cost, or delivery risk.
3. Incorporate user feedback into the design. Confirm any proposed scope changes instead of silently adopting them.
4. If the user declines further questions, make unresolved assumptions explicit in the proposal. This does not waive the approval requirement.

### Step 6: Present the High-Level Plan and Confirm

1. Present the complete proposed plan in chat before creating or editing any plan file. Cover component responsibilities, boundary and folder-structure changes, interactions, data/control flow, and integration points.
2. Explain key decisions, alternatives, trade-offs, risks, assumptions, deferred work, delivery sequence, validation, and rollout or migration needs. Keep the plan architectural rather than a file-by-file task list. Use a compact Mermaid diagram when it clarifies relationships or flow.
3. When Step 4 applies, include the official sources, relevant versions, and facts they verify.
4. Propose the destination `.backlog/technical-plan/<story-title-slug>.md`. Derive a lowercase kebab-case slug from the agreed title, remove path separators and invalid filename characters, and ask for a usable title if no meaningful slug remains.
5. Check whether the destination already exists. Never overwrite another work item's plan; agree on a more specific title and slug if there is a collision.
6. Ask the user to approve, refine, or reject the plan and destination. Incorporate feedback and re-present the revised plan until explicitly approved. If rejected or approval is absent, stop without writing files.

### Step 7: Document the Approved Plan

1. Only after explicit approval, create `.backlog/technical-plan/` if needed and write `.backlog/technical-plan/<story-title-slug>.md`, or update the existing plan the user agreed to revise.
2. Use the Plan Template below. Capture the agreed work-item context, source reference, scope, acceptance outcomes, and high-level technical decisions so the plan is understandable independently of the conversation.
3. Document the approved architecture, justified restructuring, implementation sequence, validation strategy, compatibility or rollout needs, and material risks and assumptions. Do not introduce unapproved decisions or expand into an exhaustive implementation checklist.
4. Preserve unrelated content and any existing implementation closeout when revising a plan. Leave the supplied source work item unchanged.
5. Verify the saved document matches the approved proposal and destination. Report write failures explicitly; do not claim the plan was saved if persistence failed.

### Step 8: Report Back

1. Link to the created or updated plan file.
2. Summarise the approved architecture, scope, notable trade-offs, and assumptions in 3-6 bullets.
3. Point the user at `code-story-implementer` with the standalone plan path for execution.

## Required Protocol

1. Only create or update the approved plan under `.backlog/technical-plan/`. Do not modify production code, source work items, repository configuration, or external trackers.
2. Accept any supplied work item; do not require a story under `.backlog/` or prior use of `code-story-author`.
3. Never use model memory as evidence for a version-sensitive technical fact; verify it against current official online documentation as required by Step 4.
4. Never create or update a plan file before explicit approval of the current proposal. Silence, skipped questions, or approval of an earlier version does not approve later design changes.
5. Keep the plan high-level and grounded in repository evidence. Describe components, responsibilities, contracts, and sequencing rather than enumerating file operations.

## Planning Principles

1. Choose the simplest coherent architecture that satisfies the agreed outcomes. Do not optimize for the fewest touched files at the expense of clarity or maintainability.
2. Reuse sound existing code, but do not be afraid to change code, reorganize folders, or redraw component boundaries when the work item justifies it. Explain why the change is worth its migration and compatibility costs.
3. Avoid unrelated restructuring, speculative abstractions, and unnecessary dependencies. Keep contracts and dependencies explicit.
4. Make testing, observability, error handling, security, and rollout considerations proportional to the behavior and risk.

## Plan Template

````markdown
## Work item

* Source: <Workspace-relative source link, issue URL/identifier, or user-supplied text>
* Plan status: Approved by the user

<Brief problem statement and intended outcome.>

## Technical Implementation Plan

### Architecture and approach

<Component responsibilities, interactions, data/control flow, key contracts, and any justified changes to existing code, boundaries, or folder structure.>

### Key decisions and trade-offs

<Chosen approach, material alternatives, and rationale.>

### Risks, assumptions, and deferred work

<Material risks, assumptions accepted by the user, and explicit deferrals. State when none remain.>
````

## Response Format

After updating the file, return:

* Link to the saved plan file.
* Summarise the approved approach, scope, and notable assumptions in 3-6 bullets.
* List official documentation used for version-sensitive decisions, or state that none was required.
* Point the user at `code-story-implementer` with the standalone plan path.
