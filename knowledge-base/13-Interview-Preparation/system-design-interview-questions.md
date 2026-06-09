# System Design Interview Questions

## Questions

1. How would you design a scalable AI chat API?
Answer: Clarify concurrency, latency, and cost goals first, then separate the API layer, session/cache layer, durable storage, model/provider integration, and observability stack.

2. How would you design an enterprise RAG platform?
Answer: Separate ingestion from query serving, store raw data and metadata separately, retrieve with permissions-aware filtering, rerank evidence, and generate answers with citations and refusal behavior for weak evidence.

3. How would you design a multi-agent research system?
Answer: Use explicit orchestration, constrained tools, stored intermediate state, step budgets, and final synthesis with traceability.

4. How would you reduce AI platform cost without destroying UX?
Answer: Cache repeated work, choose smaller models for simple tasks, reserve expensive models for hard cases, and monitor token and latency budgets.

5. What would you monitor first after launch?
Answer: Request volume, first-token latency, total latency, provider failures, cache hit rate, retrieval quality, and cost per request.

## Related Topics

- [backend-and-ai-scenarios.md](../05-System-Design/backend-and-ai-scenarios.md)
- [system-design-interview-questions.md](../05-System-Design/system-design-interview-questions.md)
