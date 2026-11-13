---
description: "Use when designing, extending, or refactoring code. Covers focused responsibilities, explicit contracts, caller-supplied configuration, dependency injection, orchestration, persistence, and evolving abstractions while keeping solutions clear, complete, and minimal."
applyTo: "**/*.py,**/*.pyi,**/pyproject.toml,**/uv.lock"
---

# Python Coding Standards

## Python Environment And Dependencies

- Use `uv` for Python package and environment management. Do not invoke `pip`, Poetry, Pipenv, or Conda directly.
- Reuse the target repository's root `.venv/`; do not create another virtual environment.
- Add project dependencies with `uv add <package>` or `uv add --group <group> <package>` so `pyproject.toml` remains authoritative.
- Install an intentionally untracked, non-project package with `uv pip install <package>`.

## Docstrings And Comments

- Add concise Google-style docstrings to modules, classes, public functions, and methods. Document constructors and private helpers when their contracts or intent are not evident.
- Start with a short summary. Include `Args:`, `Returns:`, `Yields:`, `Raises:`, `Attributes:`, and `Examples:` only when applicable.
- Keep each docstring section entry on one physical line. Write descriptions as prose without backticks, snippets, or cosmetic line wrapping.

## Formatting, Naming, And Imports

- Use descriptive snake_case names for functions, methods, variables, filenames, and modules; PascalCase for classes; and uppercase names for constants.
- Limit filenames and module names to four words or fewer while clearly conveying their purpose.
- Group imports into two blocks separated by a blank line: standard-library imports first, followed by imports from your own application package.

## Types And Models

- Annotate parameters and return values, including `-> None` where applicable. Use built-in generics and union syntax when supported by the minimum runtime.
- Use appropriate collection abstractions from `collections.abc` for inputs and concrete collection types for returned values.
- Prefer precise types over `Any`. Reserve `Any` for genuinely heterogeneous values or third-party boundaries, and explain that boundary when it is not evident.
- Use enums for named finite choices and dataclasses for shared structured results. Prefer frozen dataclasses when fields should not be reassigned; freezing does not make nested collections immutable.
- Use validated models, such as Pydantic models, when external definitions require structural and value validation.