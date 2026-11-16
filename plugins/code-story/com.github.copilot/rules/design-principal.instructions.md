---
description: "Use when designing, extending, or refactoring code. Covers single-responsibility methods, decomposition, reuse, cohesive package and folder boundaries, explicit contracts, caller-supplied configuration, dependency injection, orchestration, persistence, and evolving abstractions while keeping solutions clear, complete, and minimal."
---

# Design Principles

Prefer standalone classes, functions, and modules with explicit responsibilities, dependencies, and contracts. Let callers configure and compose them rather than hiding setup, orchestration, or persistence inside processing components.

## Keep Components Focused

- Give each component one cohesive job. Keep transformation, validation, provider calls, and storage independently usable rather than tying them to a particular workflow.
- Make accepted inputs, returned results, failure outcomes, ownership, and permitted side effects explicit in public contracts.
- Leave workflow coordination to the caller as much as possible. A dedicated orchestrator may coordinate collaborators, but should delegate their implementation details.
- Processing components should not acquire files, discover environment settings, or provision resources unless that is their explicit job.
- Actions return results without choosing a store or saving those results. The caller or orchestrator decides whether, when, and where to save through an injected storage implementation. Storage components persist data because that is their explicit job.
- Preserve required transactional consistency. The caller or orchestrator coordinates the transaction when operations must succeed or fail together.

## Keep Methods Small And Single-Responsibility

- Give every function and method one cohesive responsibility at a consistent level of abstraction. A short method can still mix responsibilities.
- Break complex methods into small, clearly named helpers for distinct responsibilities or nontrivial steps. Keep straightforward one- or two-line operations inline when extraction would only add indirection; do not split solely to meet a line-count limit.
- Design with reuse in mind. Extract genuinely shared mechanics, but keep different domain rules in separately named operations that compose those helpers. Do not force reuse by merging unrelated behavior into a generic method.

## Organize Packages And Folders Cohesively

- Create subpackages and folders when they clarify responsibilities and boundaries. Do not avoid useful structure merely to minimize the number of directories.
- Give each package or folder a clear purpose and meaningful name. Group classes and modules that share a domain concern or architectural role. Use `utils`, `helpers`, or `common` containers whenever they make sense for cohesive, reusable functionality, but do not use them as catch-alls for unrelated responsibilities.
- Keep package boundaries explicit. Depend on other packages through their intended public contracts rather than reaching into implementation details, and avoid circular dependencies.
- Follow the project's existing organization and introduce only meaningful levels of structure. Avoid speculative layers or folders that add navigation without improving cohesion or separation of responsibilities.

## Let Callers Configure And Connect Components

- Pass configuration and dependencies through constructor parameters for classes or explicit parameters for functions. Use method parameters for per-operation inputs.
- Document safe defaults where appropriate and allow caller overrides. Require essential inputs rather than silently discovering them.
- Let the caller or application setup code construct and wire implementations and configure authentication, endpoints, API versions, deployment selection, retries, timeouts, and client lifecycle.
- Inject configured clients and collaborators. Consumers do not construct them or silently reconfigure or close supplied clients.
- An adapter may create clients when their setup and lifecycle are its explicit job. It accepts the required configuration from the caller and makes ownership and cleanup explicit.

## Use Abstractions Deliberately

- Define small interfaces or protocols for concrete external boundaries or stable jobs with interchangeable behavior. For storage, inject a service-specific adapter through a storage contract; for alternative validation strategies, inject a validator through a validation contract.
- Revisit designs as requirements evolve. When real variation emerges, refactor existing code and callers around an appropriate shared contract alongside the new implementation. Do not preserve an unsuitable design just to avoid editing existing code; keep changes relevant to the requirement and verify intended behavior is preserved.
- Search for existing helpers, models, and interfaces before adding new ones. Reuse them where they fit, without hiding meaningful differences or grouping unrelated responsibilities.
- Keep specific parsing, mapping, and object-to-object conversion logic concrete unless interchangeable conversion behavior is an actual requirement.
- Prefer composition over implementation inheritance. Use interfaces or protocols for contracts; reserve implementation inheritance for genuine, substitutable specialization.
- A dispatcher may select among injected implementations when selection is its explicit job. A map of operation names to supplied implementations is appropriate; avoid hidden global registration and service locators.
- Prefer the least total code needed for a clear, correct, complete solution, not the smallest diff or compressed syntax. Avoid speculative abstractions, splitting trivial operations solely for architectural purity, and unnecessary factories, configuration layers, wrappers, or inheritance hierarchies.

## Keep Behavior Predictable

- Make mutations and side effects intentional. Do not modify caller-owned inputs or shared state unless the contract explicitly permits it.
- Distinguish absence from falsiness with checks appropriate to the language and contract. Preserve valid zero, false, null, and empty values.
- Keep failure, no-match, skipped work, and success distinct when they represent different outcomes.
- Make defaults and fallbacks explicit parts of the contract, not hidden recovery from programming or provider errors.