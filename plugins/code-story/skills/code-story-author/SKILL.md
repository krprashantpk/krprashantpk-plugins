---
name: code-story-author
description: Use when the user wants to create a new backlog story, capture a feature/refactor idea, draft user-facing intent, or write the description and acceptance criteria for a piece of work. Produces ONLY the story background (Description + Acceptance Criteria) under .backlog/work-items/. Does NOT plan or implement code. Triggers include "create a story", "add a backlog item", "draft a UC", "capture this idea". For technical breakdown use the code-story-technical-plan skill; for execution use the code-story-implementer skill.
---

# CodeStory Author

Author the *background* of a backlog story: business context, scope, and acceptance criteria. This skill is project-agnostic and writes ONLY the non-technical part of the story. 

Given a short user request, this skill:

1. Runs a mandatory clarifying-question loop until business intent, requirments and any ambiquity is fully understood.
2. Writes a story file at `.backlog/work-items/<ID>-<slug>.md` containing only:
    * `# <ID> - <Short Title>`
    * `## Metadata` (Parent Key, Type, Fix Version, Labels, Story Points, Start Date, Due Date — placeholders if not provided)
    * `## Description` (with optional `### Assumptions`)
    * `## Acceptance Criteria`
3. Ensures `.backlog/work-items/` exists and its parent `.backlog/` is gitignored.

## Prerequisites

* The repository has (or can have) a `.backlog/work-items/` directory relative to the repository root.
* `.gitignore` is writable.

## Required Steps

### Step 0: Honor Loaded Instructions (Mandatory)

Treat all repository instructions automatically supplied by Copilot as binding for the rest of this skill. If an instruction contradicts a step below, the instruction wins.

### Step 1: Setup

1. Ensure `.backlog/work-items/` exists relative to the repository root; create it and any missing parent directory.
2. Ensure `.backlog/` is listed in `.gitignore`; append the entry if missing.
3. Detect repo conventions for story id format by scanning `.backlog/work-items/`, existing stories elsewhere under `.backlog/`, and any sibling stories folder. Match whatever pattern exists (`UC<N>`, `STORY-<N>`, etc.). Default to `STORY-<N>` if no precedent is found.

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

1. Pick the story id and a kebab-case slug. Filename: `.backlog/work-items/<ID>-<slug>.md` (for example `UC7-short-title.md` or `STORY-12-short-title.md`).
2. If a file with that id already exists, edit it instead of creating a duplicate.
3. Write the file using the *Story Background Template* below. Include only the sections in that template — no Technical Implementation Plan, no Implementation Summary, no Closing Comment.
4. Populate the `## Metadata` block:
    * **Parent Key** — use the user-provided parent issue key if given; otherwise leave the value empty.
    * **Type** — use the user-provided value if given; otherwise leave the value empty.
    * **Fix Version** — use the user-provided value if given (for example `2026.05`, `v1.4.0`, `Sprint 23`); otherwise leave the value empty.
    * **Labels** — comma-separated list if the user provided any (for example `backend, refactor`); otherwise leave the value empty.
    * **Story Points** — numeric value if the user provided one; otherwise leave the value empty.
    * **Start Date** — use the user-provided value if given (for example `2026-08-04`); otherwise leave the value empty.
    * **Due Date** — use the user-provided value if given (for example `2026-08-05`); otherwise leave the value empty.
    * Do not invent or estimate these values. Surface them as quick prompts to the user during the clarifying loop, but accept that they may stay unset.
5. Description should be 1-3 short paragraphs in the user's voice, framed in business terms.
6. Acceptance Criteria must be observable and testable. Avoid implementation language ("uses table X", "calls service Y").
7. Add `### Assumptions` only if there are unconfirmed items.

### Step 4: Report Back

1. Print the workspace-relative path to the story file.
2. Summarise the story in 2-4 bullets.
3. Remind the user that `code-story-technical-plan` creates a separate approved high-level plan under `.backlog/technical-plan/`, and `code-story-implementer` executes that plan and records its implementation closeout.

## Required Protocol

1. Step 2 is mandatory and must precede any drafting unless the user explicitly opts out.
2. Never write story files outside `.backlog/work-items/` or modify files other than stories and `.gitignore` without explicit user approval.
3. Never include a Technical Implementation Plan, file lists, code, signatures, or implementation notes in this skill's output. Keep the story strictly at the business/UX level.
4. Prefer editing an existing story over creating duplicates.

## Story Background Template

````markdown
# <ID> - <Short Title>

## Metadata

- **Parent Key:**
- **Type:**
- **Fix Version:**
- **Labels:**
- **Story Points:**
- **Start Date:**
- **Due Date:**

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
* Point the user at `code-story-technical-plan` to collaboratively develop and approve a standalone high-level plan for the story.
