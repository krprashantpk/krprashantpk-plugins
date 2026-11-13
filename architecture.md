---
title: Architecture
description: "Marketplace discovery and the CodeStory search, export, planning, implementation, and closeout workflows."
---

## Architecture

This repository is a GitHub Copilot agent-plugin marketplace. The marketplace manifest discovers independently installable plugins under `plugins/`; each plugin manifest exposes a directory of self-contained skills. CodeStory also exposes bundled hosted MCP server definitions.

```mermaid
flowchart TD
    User[Copilot user] --> Marketplace[Marketplace manifest]
    Marketplace --> CodeShift[code-shift plugin]
    Marketplace --> CodeStory[code-story plugin]
    CodeShift --> MigrationSkills[Migration skills]
    CodeStory --> MCP[Bundled MCP definitions]
    MCP --> Client[Copilot client: server enablement and OAuth]
    Client --> Hosted[GitHub, Azure DevOps, and Atlassian hosted MCP servers]
    CodeStory --> Search[CodeStory work item search]
    Search --> SearchReference[Selected provider search reference]
    SearchReference --> ReadOnly[Read-only search or exact lookup]
    ReadOnly --> Chat[Results and requested metadata/comments in chat only]
    CodeStory --> Author[CodeStory author]
    CodeStory --> Export[CodeStory export]
    CodeStory --> Plan[CodeStory technical plan]
    CodeStory --> Implement[CodeStory implementer]
    Author --> Story[.backlog/work-items/ story]
    Story --> Export
    Export --> Setup[Official CLI setup and field mapping]
    Setup --> Preview[Preview and explicit approval]
    Preview --> Provider[GitHub, Jira Cloud, or Azure DevOps Services]
    Provider --> Verified[Read back and verify remote item]
    Verified --> Links[Append remote URL to local story]
    Story --> Plan
    WorkItem[Any supplied work item] --> Plan
    OfficialDocs[Current official documentation] --> Plan
    Plan --> Approval[User feedback and explicit approval]
    Approval --> PlanFile[.backlog/technical-plan/ standalone plan]
    PlanFile --> Implement
    RepoRules[Target repository instructions] --> Implement
    Implement --> Code[Code, tests, and documentation]
```

CodeStory's [MCP configuration](plugins/code-story/.mcp.json) contains no credentials or organization-specific settings. The client owns connections and authentication; callers supply confirmed provider scope, including the organization on each Azure DevOps tool call. Users can disable individual bundled servers to retain existing global connections without disabling the skills. These definitions do not change the search and export skills' CLI-first behavior. See [MCP setup](plugins/code-story/README.md#mcp-servers) for requirements and verification.

The independent search skill accepts keywords across title, description, and acceptance criteria, or exact IDs/keys and URLs. Provider references own authentication, field discovery, queries, and bounded pagination for Azure DevOps Services, GitHub issues, and Jira Cloud. Results remain in chat, with comments only on request; search never writes backlog files or modifies remote items. Access failures and partial field/search coverage are explicit.

The optional exporter guides CLI installation and authentication, resolves destination fields with the user, and creates an item only after preview approval. It appends the verified browser URL to the local story without changing its existing content. Partial failures are reported explicitly and recover the existing remote item rather than creating a duplicate.

The planner accepts work items from any source without requiring the author skill. It develops a high-level architectural approach with the user, including justified code or folder restructuring, and writes the standalone plan only after explicit approval. The source work item remains unchanged.

The implementer consumes the approved high-level plan, translates its architecture and delivery sequence into concrete changes, and appends implementation status and closeout to the plan. Legacy stories with embedded plans remain supported. Implementation follows the target repository's instructions, language, frameworks, and tooling without bundled cross-project defaults.

Material design or scope changes return to planning for approval. Required checks and in-scope acceptance outcomes must pass before implementation is recorded as complete; unavailable or failing validation remains explicit in an incomplete or blocked closeout.
