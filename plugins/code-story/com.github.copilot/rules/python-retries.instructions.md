---
description: "Use when generating, modifying, or reviewing Python external-service operations. Guide retry eligibility, SDK-native ownership, additive-jitter or chained backoff, proposed defaults, per-method approval, sync and async execution, and explicit failures."
applyTo: "**/*.py"
---

# Python Transient-Failure Retries

Apply the following workflow to each new or modified external-service operation. Do not retrofit untouched integrations or add retries to unrelated local logic. Obtain the per-operation approval described below before introducing or changing retry behavior.

## Retry Eligibility And Operation Safety

### Transient-Failure Classification

- Retry only documented transient conditions relevant to the operation, such as qualifying temporary connection failures, throttling, or temporary service unavailability.
- Fail immediately for permanent errors, including invalid input, invalid credentials, insufficient permissions, and programming errors. Use the service's error contract rather than a blanket list of HTTP status codes.
- Treat unknown failures as non-retryable unless documentation establishes otherwise. Do not infer retryability from exception-message text or retry every exception.

### Repeat Safety And Request Replay

- Establish repeat safety before retrying side effects. A timeout can occur after the server committed a write; it does not prove that the operation was never applied.
- Use supported idempotency keys, deduplication, conditional operations, or an equivalent documented guarantee where needed. Keep the logical operation identity and idempotency key unchanged across attempts.
- If duplicate effects cannot be prevented, explain the risk and do not add automatic retries.
- Retry the smallest safe operation, not a sequence that would repeat completed side effects. Prepare replayable inputs; do not reuse an exhausted stream or partially consumed request without a supported reset.

## Retry Ownership And Library Selection

- Inspect existing retries across the call chain, shared clients, transports, and relevant job or workflow redelivery. Check the installed SDK version's official documentation for defaults, eligible failures, attempt counters, waits, timeouts, and server-hint handling.
- Prefer SDK-native retries when they satisfy the confirmed policy. Translate total attempts into the SDK's documented configuration; a setting named retries may exclude the initial call.
- When SDK-native retries are unavailable or insufficient, use [Tenacity](https://tenacity.readthedocs.io/en/latest/) instead of a handwritten retry loop.
- Before making Tenacity the owner, disable overlapping SDK or transport retries for that operation through supported settings. Do not wrap an already-retrying call or nest retry decorators.
- If overlapping retries cannot be disabled safely, explain the conflict and request another approved approach. Coordinate unavoidable workflow redelivery with the attempt budget and repeat-safety contract rather than multiplying attempts silently.
- Reuse existing project helpers and conventions. Follow the bundled [Python dependency-management instructions](python.instructions.md) when Tenacity is needed; do not add it when SDK retries suffice.

## Backoff Strategy Selection

### Exponential Backoff With Additive Jitter

- Recommend this strategy for general external-service recovery when progressively longer waits are appropriate.
- For Tenacity, the proposed configuration is `wait_exponential_jitter(initial=5, jitter=2, max=30)`.
- With the default exponential base of 2, the delay before retry number `n`, starting at 1, is `min(5 * 2 ** (n - 1) + uniform(0, 2), 30)` seconds. This produces 5-7, 10-12, and 20-22 seconds, then 30 seconds if further attempts are approved.
- The maximum caps the entire delay, including jitter. Once the exponential component reaches the cap, additive jitter no longer spreads the retries.

### Chained Backoff With Jitter

- Recommend this strategy when an explicitly staged recovery schedule is required rather than exponential growth.
- For Tenacity, the proposed configuration is `wait_chain(wait_fixed(5), wait_fixed(10)) + wait_random(0, 2)`.
- This produces a 5-7 second first wait and a 10-12 second second wait. With the proposed three total attempts, there are only two waits.
- `wait_chain` repeats its final stage if additional attempts are approved. Adding `wait_random` supplies jitter; a chain alone is neither inherently exponential nor jittered.
- Keep each complete delay, including added jitter, within the approved maximum. Reconcile stage values and limits whenever the policy changes.

### Workload-Specific Alternatives

- Do not silently substitute another strategy. For high-contention workloads, explain the clustering risk of additive jitter and propose full jitter with `wait_random_exponential` when appropriate.
- Full jitter samples within a widening window; a multiplier of 5 does not guarantee a five-second minimum. Confirm the actual delay semantics of any alternative.

## Policy Defaults And Per-Operation Approval

### Proposed Starting Values

These are recommendations for discussion, not preapproved settings or universal latency requirements.

| Policy Setting | Proposed Value |
| --- | --- |
| Total attempts | 3: the initial call plus at most 2 retries |
| Preferred backoff | Exponential with additive jitter |
| Initial exponential delay | 5 seconds |
| Exponential base | 2 |
| Additive jitter | Uniformly random from 0 to 2 seconds |
| Maximum backoff wait | 30 seconds, including jitter |
| Chained alternative | 5 seconds, then 10 seconds, each with 0-2 seconds of jitter |
| Per-attempt timeout | Propose an operation-appropriate value from documented timeout semantics and latency requirements |
| Overall time budget | Propose an operation-appropriate value covering execution, waits, and retry overhead |
| Server retry hints | Honor documented valid hints; stop if the required delay exceeds the approved wait limit or remaining budget |

### Approval Boundary And Configuration Isolation

- Before adding or changing retries on each operation or method, present the operation, retry owner, documented eligible failures, repeat-safety guarantees, selected strategy, and every setting above. Let the end user confirm or override them explicitly.
- Approval is required for configuring SDK-native retries and for introducing a client whose defaults enable retries. Approval of one method does not authorize another, even when their policies match.
- If confirmation is absent, unavailable, or declined, preserve that method's existing retry behavior and explain what remains blocked.
- Do not change shared-client settings in a way that affects unapproved methods. Use supported operation-level configuration or suitable isolation; otherwise request approval for every affected method.

## Synchronous And Asynchronous Execution

### Retry Policy Configuration

- Configure an explicit narrow retry predicate, the approved finite attempt limit, and the selected bounded wait strategy.
- For Tenacity exception-based retries, use `retry_if_exception_type` with specific documented transient exception types, or `retry_if_exception` with a predicate checking documented error details. Use `stop_after_attempt` with the approved total count.
- Do not use bare `@retry`, Tenacity's broad default predicate, `retry_if_exception_type(Exception)`, or broad catch-and-retry handlers.
- If retry eligibility is expressed through returned results, use an explicit documented result predicate and preserve an explicit failure outcome on exhaustion.

### Synchronous Operations

- Use the synchronous SDK with Tenacity's `@retry(...)` decorator or `Retrying(...)` controller when Tenacity is the approved owner.
- Keep the retry boundary around the smallest repeat-safe call. Use a retry controller for a code block only when every operation in that block can safely repeat.

### Asynchronous Operations

- Use the native asynchronous SDK with coroutine-aware `@retry(...)` or `AsyncRetrying(...)` when Tenacity is the approved owner.
- Await I/O and use asynchronous retry waits. Do not call blocking sleep or wrap blocking I/O in an async declaration and present it as non-blocking.
- Propagate cancellation during both execution and waiting. Never classify normal cancellation, `KeyboardInterrupt`, or `SystemExit` as transient or catch `BaseException` to retry them.

### Timeout And Deadline Enforcement

- Establish one monotonic deadline for the logical operation. Include execution, waits, and retry overhead; never reset the overall budget for each attempt.
- Verify whether SDK timeouts bound the complete attempt or only connection phases or socket inactivity. Setting a timeout to the remaining budget does not necessarily enforce a strict overall deadline.
- Use supported total-operation deadlines or cancellation where needed and safe. Bound each attempt by its approved timeout and remaining budget using the client's documented semantics. If the approved deadline cannot be enforced safely, disclose the limitation and obtain approval for a revised approach.
- Recheck the budget before waiting and immediately before every attempt, including the first. Stop if the budget is exhausted or the next required wait cannot fit; do not begin an attempt at or after the deadline.
- Do not claim that `stop_after_delay`, an attempt limit, or a retry decorator interrupts a running call or alone guarantees a strict deadline.

### Service-Directed Retry Delays

- Honor valid service retry hints, including documented `Retry-After` delay or date forms. Prefer the SDK's supported handling when it satisfies the policy.
- Never shorten a server-required delay to the backoff cap or randomize it downward. If it exceeds the approved wait limit or cannot fit the remaining budget, stop and surface the failure.
- For absent or invalid hints, follow the documented service behavior and confirmed backoff policy. Do not turn parsing failures into immediate retry storms.

## Failure Propagation And Observability

### Terminal Failure Handling

- Preserve the underlying failure and established application error contract on exhaustion. For exception-based Tenacity retries, use `reraise=True` unless the repository explicitly requires a different failure contract.
- Surface timeout or budget exhaustion explicitly through the established error contract, preserving the last underlying service failure when one exists.
- Do not swallow failures, return success-shaped defaults, or use `retry_error_callback` to disguise exhaustion as success.

### Retry Diagnostics

- Follow the bundled [logging and exception rules](logging-and-exceptions.instructions.md) and reuse existing application instrumentation.
- Include safe operation identifiers, failure category, attempt and total-attempt limit, planned delay, and available correlation identifiers. Keep logical correlation stable across attempts.
- Record routine retries at the existing appropriate level, normally `debug`; do not turn every retry into a warning or duplicate the owning boundary's terminal-error log.
- Use retry callbacks only with safe metadata. Do not assume generic exception-logging callbacks are privacy-safe or expose payloads, credentials, sensitive URLs, exception messages, or tracebacks without required redaction.

## Policy Verification

### Application Behavior Tests

- Use existing test tooling, fake or stub service responses, controlled clocks, and replaceable waits. Do not make live service calls or spend real backoff intervals to test retries.
- Test the implemented retry owner: transient failure followed by success, permanent failure on the first attempt, exhaustion at the exact approved total-call limit, and preservation of the failure contract.
- Verify the selected exponential or chained schedule, jitter bounds, and the complete delay cap without expecting exact randomized waits. Check final-stage reuse or capped behavior when the approved attempt count reaches it.
- Test elapsed-budget exhaustion, documented per-attempt timeout behavior, no attempts at or after the deadline, and server hints that either fit or require stopping.
- For applicable operations, verify non-blocking async waits, cancellation during execution and waiting, stable idempotency across ambiguous write outcomes, and unsafe writes not retried.
- Verify configuration isolation and the absence of overlapping retry layers: changing one method must not change unrelated operations.

### Approval And Change Review

- Verify through the conversation's approval record and the proposed changes, not application unit tests, that each affected method has explicit approval for its complete policy.
- Confirm that unapproved or declined operations retain their existing retry behavior and that shared-client changes do not cross the approval boundary.
