# Production AI Platform

## Goal

Combine API delivery, retrieval, agents, observability, CI/CD, and cost controls into a single production-grade platform project.

## Architecture

```text
[Users]
  |
  v
[API Gateway / FastAPI]
  |
  +--> [Auth + Rate Limits]
  +--> [Chat / RAG / Agent Services]
  +--> [PostgreSQL]
  +--> [Redis]
  +--> [Vector DB]
  +--> [Background Workers]
  +--> [Monitoring + Logs + Traces]
```

## Implementation Steps

1. Expose chat, retrieval, and agent workflows as separate service layers.
2. Add PostgreSQL for metadata and Redis for fast-path coordination.
3. Introduce background workers for ingestion, embeddings, and heavy jobs.
4. Add CI/CD, containerization, and environment management.
5. Instrument logs, metrics, traces, and cost dashboards.

## Interview Talking Points

- Why online and offline workloads should be separated
- Where to put caching and why
- How to monitor latency, cost, and failure rate together
- How to choose between direct LLM calls, RAG, and agents per request type

## Production Considerations

- SLOs for latency and availability
- Provider fallback strategy
- Data retention and PII controls
- Load testing and autoscaling

## Related Topics

- [cloud-ai-architecture.md](../06-Software-Architecture/cloud-ai-architecture.md)
- [observability-and-monitoring.md](../06-Software-Architecture/observability-and-monitoring.md)
- [cicd-for-ai-systems.md](../06-Software-Architecture/cicd-for-ai-systems.md)
