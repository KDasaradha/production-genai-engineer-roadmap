# Backend and AI Scenarios

## Purpose

This file centralizes scenario-style prompts and design anchors that were scattered across the interview and project notes.

## Scenario 1: Scale a FastAPI AI Chat API

### Problem

Users stream responses from an LLM-backed chat API. Traffic rises from hundreds to tens of thousands of concurrent users.

### Design Points

- Stateless FastAPI app behind a load balancer
- Redis for session cache, rate limits, and short-lived chat state
- PostgreSQL for durable metadata and audit records
- Background workers for slow ingestion, embedding generation, and file parsing
- Streaming responses over SSE or WebSockets
- Observability for latency, token usage, and model/provider failures

### Tradeoffs

- WebSockets are flexible but operationally heavier than SSE.
- Storing full chat history in Redis is fast but usually not durable enough alone.
- Streaming improves UX but complicates retries and partial-failure handling.

## Scenario 2: Design an Enterprise RAG Platform

### Problem

A company wants document question answering with citations, access control, and low hallucination rate.

### Design Points

- Ingestion pipeline for OCR, parsing, chunking, embedding, and metadata extraction
- Object storage for raw files
- PostgreSQL for users, documents, permissions, and ingestion jobs
- Vector database for semantic retrieval
- Reranker before final context assembly
- LLM answer generation with citations and refusal behavior on weak evidence

### Failure Modes

- Wrong chunking harms retrieval
- Weak metadata filters leak irrelevant content
- Stale embeddings after document updates
- RAG still hallucinates when retrieval is weak

## Scenario 3: Build a Multi-Agent Research System

### Problem

The system must plan, search, read, synthesize, and produce a final report from a research question.

### Design Points

- Planner node creates subtasks
- Search tool fetches sources
- Reader tool extracts and summarizes
- Memory store saves intermediate findings
- Orchestrator controls loop limits and tool permissions
- Final writer assembles output with citations

### Tradeoffs

- Agents improve coverage for multi-step tasks but increase latency and failure surface.
- Deterministic workflows are easier to test than unconstrained autonomous loops.

## Scenario 4: Rate Limit and Protect an AI API

### Problem

An expensive AI endpoint is being abused by repeated requests and prompt injection attempts.

### Design Points

- Redis sliding-window or token-bucket limiter
- Per-user quotas and API keys
- Prompt injection guardrails and content filtering
- Request logging with request IDs and user attribution
- Budget limits and fallback models for cost control

## Interview Answer Pattern

Use this structure for scenario questions:

1. Clarify scale, latency, and correctness requirements.
2. Separate online path from offline/background path.
3. Choose storage by access pattern, not by habit.
4. Explain failure modes and observability.
5. State the tradeoffs you are accepting.

## Related Topics

- [portfolio-projects.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/portfolio-projects.md)
- [enterprise-rag-platform.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/enterprise-rag-platform.md)
- [multi-agent-research-system.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/multi-agent-research-system.md)
- [interview-prep.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/interview-prep.md)
