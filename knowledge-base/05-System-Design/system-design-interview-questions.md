# System Design Interview Questions

## Core Questions

1. How would you design a scalable FastAPI AI chat API?
Answer: Separate the synchronous request path from background jobs, use FastAPI for the API layer, Redis for rate limits and short-lived state, PostgreSQL for durable metadata, and streaming for user responses. Add observability for latency, token usage, and provider failures.

2. How would you design a document question-answering system with citations?
Answer: Use an ingestion pipeline for parsing, chunking, and embeddings; store raw files in object storage, metadata in PostgreSQL, vectors in a vector DB, and retrieve then rerank before generation. Enforce access control at retrieval time.

3. How would you scale retrieval for millions of documents?
Answer: Partition ingestion from query serving, use metadata filters, maintain embedding refresh jobs, choose the right index strategy in the vector store, and add caching for repeated queries and chunk lookups.

4. How would you control cost in a production AI platform?
Answer: Cache repeated work, choose smaller models when acceptable, gate expensive features, log token usage, add quotas, and keep fallback model paths.

5. When would you avoid an agent architecture?
Answer: Avoid agents when a deterministic workflow with fixed steps is enough, because agents add latency, cost, orchestration complexity, and harder debugging.

## Common Follow-Ups

- How do you handle retries and idempotency?
- Where do you store chat history and why?
- How do you protect against prompt injection?
- What do you monitor first in production?
- How do you degrade gracefully when the model provider is down?

## Answer Pattern

Use:

```text
Requirements -> Components -> Data Flow -> Failure Modes -> Tradeoffs -> Monitoring
```

## Related Topics

- [backend-and-ai-scenarios.md](backend-and-ai-scenarios.md)
- [interview-prep.md](../13-Interview-Preparation/interview-prep.md)
