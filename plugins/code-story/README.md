# CodeStory

> A GitHub Copilot agent plugin that turns backlog ideas into planned, implemented, and verified code.

CodeStory separates backlog delivery into three explicit stages. Each stage has a focused skill with a narrow responsibility, and each resulting story remains under `.backlog/` as the durable record of intent, implementation planning, and delivery.

## Workflow

```mermaid
flowchart LR
    Idea[Idea or requirement] --> Author[Author story]
    Author --> Story[Description and acceptance criteria]
    Story --> Plan[Create technical plan]
    Plan --> Approved[Confirmed file-by-file plan]
    Approved --> Implement[Implement and validate]
    Implement --> Delivered[Code and story closeout]
```

| Stage | Skill | Outcome |
| --- | --- | --- |
| Author | `backlog-story-author` | Creates the Description and Acceptance Criteria after clarifying business intent. |
| Plan | `backlog-story-technical-plan` | Explores the repository, verifies version-sensitive facts against current official documentation, and adds a confirmed file-by-file Technical Implementation Plan. |
| Implement | `backlog-story-implementer` | Executes the plan using repository-specific rules and bundled implementation defaults, validates the result, and records the implementation summary and closing comment. |

## Installation

```sh
/plugin install code-story@krprashantpk-plugins
```

## Usage

Describe the stage you want to run in natural language. The matching skill activates automatically.

- *"Create a backlog story for exporting invoices."*
- *"Add a technical implementation plan to STORY-12."*
- *"Implement STORY-12 and close it out."*

## Project structure

```text
code-story/
├── plugin.json                                  # Plugin manifest and skills path
├── README.md                                    # Plugin overview, installation, and usage
└── skills/
    ├── backlog-story-author/
    │   └── SKILL.md                             # Authors story background and acceptance criteria
    ├── backlog-story-technical-plan/
    │   └── SKILL.md                             # Produces the confirmed file-by-file plan
    └── backlog-story-implementer/
        ├── SKILL.md                             # Implements, validates, and closes the story
        └── references/
            └── python-and-project-conventions.md # Python and project implementation defaults
```

## License

[MIT](../../LICENSE)