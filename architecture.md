# Architecture

## Architecture

This repository is a GitHub Copilot agent-plugin marketplace. The marketplace manifest discovers independently installable plugins under `plugins/`; each plugin manifest exposes a directory of self-contained skills.

```mermaid
flowchart TD
    User[Copilot user] --> Marketplace[Marketplace manifest]
    Marketplace --> CodeShift[code-shift plugin]
    Marketplace --> CodeStory[code-story plugin]
    CodeShift --> MigrationSkills[Migration skills]
    CodeStory --> Author[Backlog story author]
    CodeStory --> Plan[Backlog story technical plan]
    CodeStory --> Implement[Backlog story implementer]
    Author --> Story[.backlog story]
    Story --> Plan
    OfficialDocs[Current official documentation] --> Plan
    Plan --> Implement
    RepoRules[Target repository instructions] --> Implement
    Defaults[Bundled implementation conventions] --> Implement
    Implement --> Code[Code, tests, and documentation]
```

The implementer resolves target-repository instructions before the bundled conventions. The bundled reference supplies defaults for Python tooling, documentation, typing, logging, design, precision, and project documentation only where the target repository does not define a conflicting rule.