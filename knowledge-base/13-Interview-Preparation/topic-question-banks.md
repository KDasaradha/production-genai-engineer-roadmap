# Topic Question Banks

## Python

1. Why is AsyncIO useful in FastAPI systems?
Answer: FastAPI spends most of its time waiting on I/O such as HTTP calls, databases, Redis, or model APIs, so AsyncIO improves concurrency without blocking workers.

2. What is the difference between a generator and a normal function?
Answer: A normal function returns once and finishes, while a generator yields values lazily and preserves execution state between iterations.

3. Why are decorators useful in backend systems?
Answer: They centralize repeated cross-cutting concerns such as logging, auth, retries, timing, and caching.

## FastAPI

1. Why does FastAPI use Pydantic?
Answer: Pydantic provides validation, parsing, schema generation, and safer request/response handling.

2. Middleware vs dependency injection?
Answer: Middleware wraps the request/response lifecycle globally; dependencies inject reusable resources or checks at route level.

3. Why use `yield` in dependencies?
Answer: It supports setup and teardown patterns such as opening and closing database sessions safely.

## PostgreSQL

1. Why are composite indexes order-sensitive?
Answer: PostgreSQL can use the left-most prefix efficiently, so `(country, status)` helps `country` and `country + status` filters better than `status` alone.

2. When do you use row-level locking?
Answer: When concurrent updates to the same row can cause lost updates, overselling, or inconsistent balances.

3. What is PostgreSQL's default isolation level?
Answer: `READ COMMITTED`.

## Redis

1. Cache-aside vs write-through?
Answer: Cache-aside loads on read misses and is simpler; write-through updates cache on writes and keeps cache fresher but adds write overhead.

2. Pub/Sub vs Streams?
Answer: Pub/Sub is transient and lightweight; Streams are durable, replayable, and support consumer groups.

3. Sentinel vs Cluster?
Answer: Sentinel is for failover and high availability; Cluster adds sharding and horizontal scale.

## Generative AI

1. What is a token?
Answer: A token is a unit of text a model processes, which affects pricing, latency, and context limits.

2. Why do hallucinations happen?
Answer: Models predict plausible next tokens rather than retrieving guaranteed facts, so weak grounding, ambiguous prompts, and insufficient context can produce confident errors.

3. Temperature vs Top-P?
Answer: Both affect randomness in decoding; temperature scales probabilities, while top-p truncates the candidate set to a cumulative probability mass.

## RAG

1. Why can RAG still fail even though it reduces hallucinations?
Answer: Retrieval can miss the right evidence, pull noisy evidence, or provide insufficient context, and the generation step can still misinterpret it.

2. Why does chunk size matter?
Answer: Chunks that are too small lose context, while chunks that are too large dilute relevance and waste context window budget.

3. Why use reranking?
Answer: Reranking improves the quality of the final context set without requiring larger initial retrieval depth or larger models everywhere.

## Agentic AI

1. When should you use an agent?
Answer: Use an agent when the task genuinely needs tools, state, branching decisions, or multi-step planning.

2. What causes infinite loops in agent systems?
Answer: Weak stopping conditions, poor tool feedback, repeated failure recovery, and missing state control.

3. How do you reduce agent risk?
Answer: Restrict tool scope, add loop limits, log traces, validate tool outputs, and prefer deterministic orchestration when possible.

## System Design

1. How would you design a scalable AI chat API?
Answer: Separate the online request path from offline work, use FastAPI for the API layer, Redis for short-lived state and rate limits, PostgreSQL for durable data, background workers for ingestion, and observability for latency and cost.

2. How would you control AI cost in production?
Answer: Cache repeated work, use smaller models where possible, add retrieval before generation, track token usage, and apply quotas and fallbacks.

## Related Topics

- [interview-prep.md](interview-prep.md)
- [backend-and-ai-scenarios.md](../05-System-Design/backend-and-ai-scenarios.md)
