# Python And Project Implementation Conventions

Use these conventions as implementation defaults. Read the target repository's instructions first; when they conflict with this reference, the target repository's instructions win. Apply Python-specific sections only to Python projects and Python files.

## Python Environment And Dependencies

- Use `uv` for Python package and environment management. Do not invoke `pip`, Poetry, Pipenv, or Conda directly.
- Reuse the target repository's root `.venv/`; do not create another virtual environment.
- Add project dependencies with `uv add <package>` or `uv add --group <group> <package>` so `pyproject.toml` remains authoritative.
- Install an intentionally untracked, non-project package with `uv pip install <package>`.
- Synchronize declared dependencies with `uv sync` or `uv sync --all-groups`.
- Run Python tools through the project environment with `uv run <command>` or the root virtual environment's Python executable.

## Python Documentation And Types

- Add Google-style docstrings to every Python module, class, function, and method, including constructors and private methods where meaningful.
- Keep the opening summary short. Use `Args:`, `Returns:`, `Yields:`, `Raises:`, `Attributes:`, and `Examples:` only when applicable.
- Keep each docstring section entry on one physical line. Write descriptions as prose without backticks, snippets, or cosmetic line wrapping.
- Add complete type annotations to every parameter and return value, including `-> None` where appropriate.
- Prefer Python 3.10+ syntax such as `list[str]` and `X | None`. Use precise collection abstractions from `collections.abc` for parameters and concrete types for returns.
- Avoid `Any` unless unavoidable, and document why it is required.

## Logging

- Use the standard-library `logging` module and declare `logger = logging.getLogger(__name__)` at module level.
- Use lazy `%` formatting with positional arguments. Do not format log messages with f-strings or `.format()`.
- Keep each logger call on one physical line and quote interpolated string values with single quotes in the message.
- Use `info` for normal progress and milestones, `warning` for recoverable degradation, `error` for handled failures, and `exception` inside exception handlers when a traceback is useful.
- Pair operation start and success messages. Include relevant identifiers and quantitative results so concurrent work remains traceable.

## Design

- Keep every class and method focused on one responsibility and split code before it becomes difficult to scan.
- Prefer asynchronous methods for I/O and synchronous methods for pure computation.
- Wrap external SDKs in clients, keep domain behavior in services, and leave entry points and handlers responsible only for parsing, orchestration, and response formatting.
- Prefer composition and explicit dependency injection over inheritance, registries, or service locators.
- Introduce protocols or abstract base classes only when multiple implementations exist or the contract meaningfully clarifies a boundary.
- Prefer keyless authentication such as managed identities and `DefaultAzureCredential`; do not introduce secrets when a keyless option exists.

## Numeric Precision

- Use `Decimal` for monetary, quantity, and measurement values. Construct values from strings rather than binary floating-point values.
- Preserve `Decimal` through models, database parameters, comparisons, and arithmetic.
- Convert to `float` only at a third-party boundary that requires it, and document the precision loss.

## Documentation Synchronization

- If root `README.md` or `architecture.md` is missing, create it before other implementation changes.
- When files, modules, classes, methods, or function signatures change, update the Architecture section and Mermaid diagrams in `architecture.md`.
- Keep the Project Structure section in `README.md` synchronized with every file and its purpose.
- Keep the project description, setup instructions, and usage examples consistent with the implemented behavior.