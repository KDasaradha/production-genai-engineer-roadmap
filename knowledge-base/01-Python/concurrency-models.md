# Concurrency Models

## Definition

This note consolidates the quick-reference backend concepts that appeared separately across the Jupyter notes: synchronous programming, asynchronous programming, concurrency, parallelism, CPU-bound work, I/O-bound work, threads, processes, multithreading, multiprocessing, the GIL, race conditions, and deadlocks.

## Why It Exists

Most backend and GenAI systems fail because developers mix up these terms and then choose the wrong runtime model, worker model, or scaling strategy.

## When To Use

- Use sync code for simple sequential workflows and scripts.
- Use async code for I/O-bound APIs, database calls, streaming, and external model requests.
- Use multiprocessing for CPU-heavy workloads such as batch embedding generation, PDF parsing, or local model preprocessing.
- Use threads for some blocking integrations when async-native libraries are unavailable.

## When Not To Use

- Do not use async for CPU-bound math-heavy loops and expect a speedup.
- Do not use multiprocessing for trivial workloads where process startup cost dominates.
- Do not use shared mutable state casually across threads or workers.

## Internal Working

| Concept | Core Idea | Best Fit | Common Trap |
| --- | --- | --- | --- |
| Sync | One task blocks the next | Small or simple flows | Poor throughput on I/O waits |
| Async | One thread cooperatively switches tasks on waits | I/O-bound APIs and streaming | Blocking the event loop with sync calls |
| Concurrency | Multiple tasks make progress in overlapping time | Chat systems, high-throughput APIs | Confusing it with true parallel CPU work |
| Parallelism | Multiple tasks run at the same time | CPU-heavy workloads | Using it for simple I/O tasks |
| Thread | Lightweight execution path inside a process | Blocking I/O integrations | GIL limits CPU scaling |
| Process | Separate OS-level worker with its own memory | CPU-heavy work and isolation | Higher memory and startup cost |

## Examples

- Sync: one request waits for the database, then the next request begins.
- Async: one request awaits the database while the server handles other requests.
- Multiprocessing: multiple workers preprocess large document batches in parallel.
- Threads: a legacy SDK performs blocking calls while the main app stays responsive.

## Common Mistakes

- Using `time.sleep()` inside `async def`.
- Calling blocking database or HTTP clients from async routes.
- Expecting multithreading to bypass the GIL for Python CPU-heavy work.
- Ignoring race conditions around counters, balance updates, or shared caches.
- Creating deadlocks by locking resources in inconsistent order.

## Best Practices

- Match the concurrency model to the workload type first.
- Keep the event loop clean by using async-compatible libraries.
- Use queues, locks, or database transactions when shared state matters.
- For CPU-heavy workloads, prefer worker processes or external task queues.
- Measure latency and throughput before changing architecture.

## Interview Questions

1. What is the difference between async, concurrency, and parallelism?
2. When should you use threads vs processes in Python?
3. What does the GIL block, and what does it not block?
4. Why is async useful for FastAPI?
5. What causes race conditions and deadlocks?

## Interview Answers

- Async is a non-blocking coordination model for I/O waits; concurrency means overlapping progress; parallelism means simultaneous execution.
- Threads are useful for some blocking I/O workloads; processes are better for CPU-heavy work because they bypass the GIL.
- The GIL limits concurrent execution of Python bytecode in a single process, but it does not prevent concurrency for I/O waits.
- FastAPI benefits from async because requests often wait on networks, databases, caches, or model APIs.
- Race conditions happen when shared state is modified without coordination; deadlocks happen when competing tasks wait on each other forever.

## Related Topics

- [asyncio.md](asyncio.md)
- [fastapi-advanced.md](../02-FastAPI/fastapi-advanced.md)
- [postgresql-advanced-patterns.md](../03-PostgreSQL/postgresql-advanced-patterns.md)
- [redis-advanced-patterns.md](../04-Redis/redis-advanced-patterns.md)
