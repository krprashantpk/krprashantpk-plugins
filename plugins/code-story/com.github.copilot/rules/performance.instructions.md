---
description: "Use whenever writing, modifying, or reviewing code. Require performant implementations, async non-blocking I/O wherever supported, deadlock prevention, bounded concurrency, and explicit flags for inefficient code."
applyTo: "**"
---

# Performance And Async Safety

## Async By Default

- Always use native async, non-blocking APIs for I/O wherever the runtime and dependencies support them, including network, database, file, and subprocess operations. Propagate async and await through the calling chain.
- Never block an event loop or UI thread with synchronous I/O, sleeps, task joins, busy-waiting, or synchronous waits on async results. Do not start a nested event loop to force an async operation into a synchronous interface.
- When a required dependency only provides blocking APIs, isolate that work using the runtime's supported executor or worker mechanism. Explain the limitation rather than presenting a blocking wrapper as non-blocking.
- Keep small, purely computational helpers synchronous. An async declaration does not make CPU work parallel; move substantial CPU-bound work off async execution threads using appropriate workers when justified.
- Run independent I/O concurrently when safe, but bound task creation, in-flight operations, queues, and buffers. Preserve ordering, transaction boundaries, and dependencies where concurrency would change behavior.
- Await every task or give it an explicit lifecycle owner that observes failures, handles cancellation, and joins it during shutdown. Prefer structured concurrency over fire-and-forget work.

## Prevent Deadlocks And Resource Starvation

- Minimize shared mutable state and keep critical sections short. Use async-aware synchronization for async tasks; never hold a blocking lock across an await, I/O operation, or external callback.
- Establish and follow one consistent acquisition order when multiple locks or scarce resources are needed. Never wait for another task whose progress depends on releasing a resource held by the waiting task.
- Avoid cyclic task dependencies, re-entering non-reentrant locks, and workers waiting for tasks queued to the same exhausted worker pool. Async locks can deadlock too; reason about dependency cycles, not just blocking APIs.
- Use explicit, configurable timeouts and cancellation for external I/O, resource acquisition, queue waits, and shutdown. Treat timeouts as failure bounds, not a substitute for fixing dependency cycles.
- Release owned locks, permits, connections, streams, and workers on success, failure, timeout, and cancellation using scoped cleanup or `finally`. Do not swallow cancellation or hide failures; leave caller-owned resources open unless the contract transfers ownership.

## Write Performant Code

- Choose algorithms and data structures appropriate to expected input sizes. Avoid unnecessary repeated scans, nested work, allocations, serialization, and recomputation.
- Avoid N+1 calls and unnecessary round trips. Use supported batching, selective queries, appropriate indexes, pagination, and streaming instead of fetching or materializing unbounded data.
- Reuse expensive clients and connection pools according to their concurrency and lifecycle contracts. Respect service limits and backpressure; do not introduce unbounded retries or parallelism.
- Add caching only for a demonstrated need, with explicit size limits, expiry or invalidation, and correctness under concurrent access.
- Preserve correctness, readability, and existing behavior. Measure suspected bottlenecks with representative workloads before adding complexity; never invent benchmark results or claim unmeasured speedups.

## Flag Inefficient Code

- Whenever you encounter less performant code while implementing or reviewing a task, explicitly flag it. Include the file or symbol, the inefficient operation, its likely impact, supporting evidence, and a concrete improvement.
- Distinguish measured bottlenecks from suspected inefficiencies. State assumptions and measurement gaps rather than presenting speculation as fact.
- Fix performance problems caused by or directly coupled to the current change. Report unrelated findings without silently expanding scope or rewriting unrelated code.

## Validate Performance And Concurrency

- Use existing tests and profiling or benchmark tooling where relevant. Check representative data sizes and concurrency levels, not only a single successful request.
- For concurrency changes, test contention, task failures, cancellation, timeouts, resource exhaustion, and shutdown. Use bounded test waits so a deadlock fails instead of hanging indefinitely, and verify cleanup and error propagation.
