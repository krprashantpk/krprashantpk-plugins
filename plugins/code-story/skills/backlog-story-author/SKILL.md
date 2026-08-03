---
name: backlog-story-author
description: Use when the user wants to create a new backlog story, capture a feature/refactor idea, draft user-facing intent, or write the description and acceptance criteria for a piece of work. Produces ONLY the story background (Description + Acceptance Criteria) under .backlog/. Does NOT plan or implement code. Triggers include "create a story", "add a backlog item", "draft a UC", "capture this idea". For technical breakdown use the backlog-story-technical-plan skill; for execution use the backlog-story-implementer skill.
---

# Backlog Story Author

Author the *background* of a backlog story: business context, scope, and acceptance criteria. This skill is project-agnostic and writes ONLY the non-technical part of the story. The Technical Implementation Plan and Implementation Summary are added later by sibling skills.

## Overview

Given a short user request, this skill:

1. Runs a mandatory clarifying-question loop until business intent is fully understood.
2. Writes a story file at `.backlog/<ID>-<slug>.md` containing only:
    * `# <ID> - <Short Title>`
    * `## Metadata` (Fix Version, Labels, Story Points — placeholders if not provided)
    * `## Description` (with optional `### Assumptions`)
    * `## Acceptance Criteria`
3. Ensures `.backlog/` exists and is gitignored.

It does *not* explore the codebase, propose file changes, or write the Technical Implementation Plan. Those belong to the `backlog-story-technical-plan` skill.

## Prerequisites

* The repository has (or can have) a `.backlog/` directory at the root.
* `.gitignore` is writable.

## Required Steps

### Step 0: Honor Loaded Instructions (Mandatory)

Treat all repository instructions automatically supplied by Copilot as binding for the rest of this skill. If an instruction contradicts a step below, the instruction wins.

### Step 1: Setup

1. Ensure `.backlog/` exists at the repository root; create it if missing.
2. Ensure `.backlog/` is listed in `.gitignore`; append the entry if missing.
3. Detect repo conventions for story id format by scanning `.backlog/` and any sibling stories folder. Match whatever pattern exists (`UC<N>`, `STORY-<N>`, etc.). Default to `STORY-<N>` if no precedent is found.

### Step 2: Mandatory Clarifying-Question Loop

This step is **required** before drafting any story content.

1. Treat the user's prompt as a seed. Identify gaps and ambiguities across these dimensions (skip a dimension only when it is genuinely irrelevant to a *business-level* story):
    * **Goal & scope** — what is in scope, what is explicitly out of scope.
    * **Users & triggers** — who or what initiates the behaviour.
    * **Inputs & outputs** — observable data going in and coming out (business-level, not schemas).
    * **Behavioural rules** — happy path, edge cases, error handling, idempotency.
    * **Non-functional** — performance, security, observability, cost, compliance expectations.
    * **Compatibility & rollout** — backwards compatibility, migration, feature flags, environments.
    * **Success measures** — what proves the story is done from a user/business perspective.
2. Ask focused questions in rounds of 3-7, grouped by theme. State any assumption you would otherwise make and ask the user to confirm or correct it.
3. After each user response, restate your current understanding in 3-6 bullets, then ask the next round on whatever is still unclear.
4. Continue until you can honestly say there are no remaining ambiguities. Only then proceed.
5. If the user explicitly says "stop asking, just draft it" (or equivalent), proceed — but record every unresolved item in `### Assumptions` so it is visible.

Do not skip this loop because the request "seems clear". The bar is: *the resulting Description and Acceptance Criteria leave no room for misinterpretation by a downstream technical-plan author.*

### Step 3: Write the Story Background

1. Pick the story id and a kebab-case slug. Filename: `.backlog/<ID>-<slug>.md` (for example `UC7-short-title.md` or `STORY-12-short-title.md`).
2. If a file with that id already exists, edit it instead of creating a duplicate.
3. Write the file using the *Story Background Template* below. Include only the sections in that template — no Technical Implementation Plan, no Implementation Summary, no Closing Comment.
4. Populate the `## Metadata` block:
    * **Fix Version** — use the user-provided value if given (for example `2026.05`, `v1.4.0`, `Sprint 23`); otherwise leave the value empty.
    * **Labels** — comma-separated list if the user provided any (for example `backend, refactor`); otherwise leave the value empty.
    * **Story Points** — numeric value if the user provided one; otherwise leave the value empty.
    * Do not invent or estimate these values. Surface them as quick prompts to the user during the clarifying loop, but accept that they may stay unset.
5. Description should be 1-3 short paragraphs in the user's voice, framed in business terms.
6. Acceptance Criteria must be observable and testable. Avoid implementation language ("uses table X", "calls service Y").
7. Add `### Assumptions` only if there are unconfirmed items.

### Step 4: Report Back

1. Print the workspace-relative path to the story file.
2. Summarise the story in 2-4 bullets.
3. Remind the user that the Technical Implementation Plan and Implementation Summary are added by the sibling skills (`backlog-story-technical-plan`, `backlog-story-implementer`).

## Required Protocol

1. Step 2 is mandatory and must precede any drafting unless the user explicitly opts out.
2. Never write outside `.backlog/` and `.gitignore` without explicit user approval.
3. Never include a Technical Implementation Plan, file lists, code, signatures, or implementation notes in this skill's output. Keep the story strictly at the business/UX level.
4. Prefer editing an existing story over creating duplicates.

## Story Background Template

````markdown
# <ID> - <Short Title>

## Metadata

- **Fix Version:**
- **Labels:**
- **Story Points:**

## Description

<Why this change exists, in business terms. 1-3 short paragraphs.>

### Assumptions

- <Anything inferred or unconfirmed; remove this subsection if there are none.>

## Acceptance Criteria

- <Observable, testable outcome>
- <...>
````

## Response Format

After writing the file, return:

* **Story path** — workspace-relative path to the created or updated file.
* **Story summary** — 2-4 bullets covering intent and key acceptance criteria.
* **Next step** — point the user at `backlog-story-technical-plan` to add the file-by-file plan.
