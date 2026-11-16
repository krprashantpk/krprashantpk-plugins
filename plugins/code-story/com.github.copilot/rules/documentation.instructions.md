---
description: "Use when creating or updating project documentation, README.md, PROJECT_STRUCTURE.md, or ARCHITECTURE.md. Covers package and application setup, environment variables, tests, Mermaid application flows, and component structure."
---

# Project Documentation

Use these instructions when creating or updating project documentation. Python docstrings and inline comments are covered by the Python instructions.

## General Rules

- Never manually word-wrap text in any Markdown (`.md`) file. Keep each prose paragraph, heading, list-item text, and table row on one physical line regardless of length; VS Code handles visual word wrapping. Preserve line breaks required by Markdown structure, code blocks, diagrams, and other whitespace-sensitive content. Do not introduce fixed-width wrapping or sentence-per-line formatting, and configure any Markdown formatter used to preserve this rule.
- Do not add YAML frontmatter or other metadata blocks at the top of project documentation unless explicitly requested. Start with a meaningful heading. Agent customization files are an exception: include the frontmatter required for their discovery and behavior.
- Keep documentation concise, practical, and grounded in the current implementation. Prefer commands, examples, tables, and diagrams over lengthy prose.
- Update affected documentation alongside changes to setup, configuration, commands, package structure, or application flow.
- Use relative Markdown links between repository documents. Do not copy machine-specific paths, credentials, or confidential data into examples.

## Required Root Documents

Keep these documents at the project root:

| Document | Required for | Purpose |
| --- | --- | --- |
| `README.md` | Every project | Overview and instructions for getting started. |
| `PROJECT_STRUCTURE.md` | Every project | Package layout and component responsibilities. |
| `ARCHITECTURE.md` | Application projects | Current application flow, primarily through Mermaid diagrams. |

- Every project has the first two documents; applications have all three. Do not create an architecture document solely to meet this baseline for a reusable package.
- Use the exact filenames and casing above. Avoid duplicate variants such as `Architect.md` and `architecture.md`; update references when renaming an existing variant.
- Link to `PROJECT_STRUCTURE.md` and, for applications, `ARCHITECTURE.md` from the README.

## README.md

Start with the project name, a brief overview of what it does, and relevant badges. Badges must reflect verified project requirements or status, not assumed versions.

For example, use this badge only when the project requires Python 3.12 or newer:

```markdown
[![Python 3.12+](https://img.shields.io/badge/python-3.12%2B-blue.svg)](https://www.python.org/downloads/)
```

### Reusable Python Packages

Include these sections in order after the overview and badges:

1. **Installation**: Supported Python versions and the actual installation command. Use the published package name or supported Git installation source; do not imply an unpublished package is available from a package index.
2. **Quick start**: A small, runnable example using the public API, with required imports, setup, and a brief explanation of the result. Make external-service requirements explicit.
3. **Contributing**: Local development setup, test commands, and how to report issues or submit changes. Link to existing contribution guidance rather than duplicating it.

Add focused feature, integration, or API-guide links when useful, without burying the installation and quick start.

### Applications And Other Projects

For projects that are not reusable Python packages, include these sections in order after the overview and badges:

1. **Prerequisites**: Use a table with columns for the tool or service, required version or access, and purpose. Include links to setup instructions where useful.
2. **Local setup**: Ordered steps to clone the project, install dependencies, and prepare required local services or resources using the project's actual tooling.
3. **Environment variables**: Use a table listing each variable's exact name, purpose, whether it is required or conditional, default if any, and a safe example. Explain where local values are set or loaded, and link to an existing example environment file when available. Never include real secrets.
4. **Running locally**: Exact startup commands, required supporting processes, and the expected local URL, endpoint, or other way to verify startup.
5. **Running tests**: Exact test commands and any required fixtures, services, or credentials. Distinguish unit tests from integration tests when their setup differs.

If no environment variables or tests exist, state that explicitly rather than inventing configuration or commands. Use additional operational documentation only where needed and link to it from the README.

## ARCHITECTURE.md

- Document the implemented application flow, not a proposed design or an exhaustive code inventory.
- Make fenced `mermaid` diagrams the primary content. Use a short introduction and brief notes only where a diagram needs clarification.
- Show the end-to-end path from the user, client, event, or trigger through processing and external dependencies to the response, output, or persistence destination.
- Label important operations and data movements. Make asynchronous queues, background workers, branches, and relevant failure or retry paths explicit when implemented.
- Use a flowchart for the main application flow. Add focused flowcharts or sequence diagrams only when distinct workflows or interactions need clarification.
- Use recognizable component names from the implementation. Keep diagrams readable by splitting complex flows instead of adding long descriptions or one oversized diagram.
- Update diagrams whenever application flow or component interactions change. Keep directory listings in `PROJECT_STRUCTURE.md` rather than duplicating them here.

## PROJECT_STRUCTURE.md

- Explain how the project is organized into packages, subpackages, and major components.
- Include a concise directory tree of meaningful source packages, tests, configuration, scripts, and documentation that actually exist. Omit generated files, caches, virtual environments, and exhaustive file listings.
- Follow the tree with a table or list of the important packages, subpackages, and components. Give each path and a brief description of its responsibility and what belongs there.
- Explain important boundaries or relationships only where they help readers locate code or decide where new code belongs.
- Keep this document synchronized with package additions, moves, renames, and removals. Do not duplicate setup instructions or detailed application-flow explanations.

## Accuracy Checks

- Verify documented commands, runtime requirements, public API examples, and environment variable names against the repository. Do not claim a command was tested unless it was run successfully.
- Check that relative links and documented paths resolve and that Mermaid diagrams accurately represent the implementation.
- Keep the required root documents consistent without repeating the same detailed content across them.
