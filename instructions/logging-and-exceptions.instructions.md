---
description: "Use when adding or reviewing Python logging, input validation, provider failures, exception translation, fallback behavior, or explicit error outcomes."
applyTo: "**/*.py"
---

# Logging And Exceptions

## Logging

- Use standard-library `logging` with `logger = logging.getLogger(__name__)` at module level. Leave handlers, levels, and destinations to the application.
- Use lazy `%` formatting with positional arguments instead of f-strings or `.format()` in logger calls.
- Use `info` for normal progress and milestones, `warning` for recoverable degradation, `error` for handled failures, and `exception` inside handlers when a traceback is useful.
- Never log sensitive information, secrets, or personally identifiable information (PII), including in exception messages or tracebacks.
- Write log messages as clear, complete sentences. Use words instead of arrows such as `->` or other symbolic shorthand; normal punctuation and technical identifiers are allowed.
- Include relevant, non-sensitive details that explain the operation, context, and outcome.
- Keep each logging call on a single line of source code.
