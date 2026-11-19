---
description: "Use when adding or reviewing Python logging, request/event lifecycle tracking, input validation, provider failures, exception translation, fallback behavior, or explicit error outcomes."
applyTo: "**/*.py"
---

# Logging And Exceptions

## Logging

- Use standard-library `logging` with `logger = logging.getLogger(__name__)` at module level. Leave handlers, levels, and destinations to the application.
- Use `info` for application startup/shutdown, meaningful processing milestones, and final request/event/job outcomes. Normal logs should explain important activity and outcomes without narrating every processing step.
- Use `debug` for routine receipt/start messages, detailed branches, per-item progress, and before/after details of external calls, persistence, or publishing. Promote these details to `info` only when they are operationally significant.
- Use `warning` for actionable recoverable degradation and `error` for failed operations requiring attention. Expected retries, validation rejections, and normal cancellation are not automatically warnings or errors.
- Reuse existing framework and tracing instrumentation instead of duplicating access logs or lifecycle messages. Aggregate repetitive progress; avoid logging every loop iteration or trivial function call.
- Log the final outcome at the owning boundary using the appropriate level. Distinguish an operation's completion from overall success, and never report success from unconditional cleanup or `finally` blocks.
- Include the operation, relevant safe target identifiers or counts, and the outcome or reason when applicable. For retries, include the attempt and limit when bounded; when recording elapsed time, use a monotonic clock.
- Use the project's existing structured logging conventions, such as `extra`, for consistent context fields. Include available correlation identifiers in related logs; establish missing correlation at the application entry point and propagate it through downstream calls and asynchronous work using existing mechanisms. Keep root correlation stable across retries while allowing distinct span and attempt identifiers.
- Use lazy `%` formatting with positional arguments instead of f-strings or `.format()` in logger calls.
- Never log sensitive information, secrets, or personally identifiable information (PII), including in structured fields, exception messages, or tracebacks. Prefer safe metadata and summaries over raw payloads, headers, or response bodies.
- Configure any required exception redaction in application-owned logging setup, covering exception messages, chained exceptions, and formatted tracebacks across all output handlers. Sanitizing only the log message is insufficient.
- Write log messages as clear, complete sentences. Use words instead of arrows such as `->` or other symbolic shorthand; normal punctuation and technical identifiers are allowed.
- Keep each logging call on a single line of source code.

## Error Handling

- Treat expected validation failures, normal cancellation, and exceptions used for normal control flow as expected outcomes, not unexpected failures requiring tracebacks.
- Ensure unexpected failures are logged once by the responsible handler or existing framework integration, with a full traceback subject to the redaction rules above. Use `logger.exception(...)` or `logger.error(..., exc_info=True)` inside the exception handler. Include the failed operation, available correlation identifiers, safe target details, and whether processing will retry, recover, or terminate.
- Never silently swallow failures or replace them with success-shaped defaults. Record operationally significant retries, timeouts, skipped work, and fallbacks with their reasons at the appropriate level; routine recovery details belong at `debug`.
- Preserve context as exceptions propagate without logging the same traceback at every layer. Use a bare `raise` when re-raising and explicit chaining (`raise ... from exc`) when translating exceptions. Do not introduce broad catch-and-continue handlers or redundant catch-and-rethrow blocks solely to add logs.
- Ensure asynchronous tasks, event consumers, and background workers surface and log failures through their owning task runner or boundary handler; do not leave task exceptions unobserved.
