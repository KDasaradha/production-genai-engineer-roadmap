# Interview Prep

## How To Prepare

For each topic, prepare three answers:

| Answer Type | Length | Use Case |
| --- | --- | --- |
| Short | 30 seconds | Recruiter or quick screening |
| Deep | 2 minutes | Technical interview |
| Production | 3-5 minutes | Senior or system design interview |

## Core Question Bank

| Area | Questions You Must Answer |
| --- | --- |
| LLMs | What are tokens? What is a context window? Why do hallucinations happen? |
| Prompting | What is few-shot prompting? How do you enforce JSON output? |
| Embeddings | What are embeddings? How does cosine similarity work? |
| Vector DBs | Why use a vector database instead of PostgreSQL only? |
| RAG | How does RAG reduce hallucination? Why can RAG still fail? |
| Agents | What is tool calling? How do you stop infinite agent loops? |
| FastAPI | How do you stream responses? How do background tasks work? |
| Production | How do you monitor latency, cost, and failure rate? |

## Model Answer Pattern

Use this structure:

```text
Definition -> Why it matters -> Example -> Tradeoff -> Production concern
```

Example:

```text
Embeddings are numeric vector representations of data. They matter because they let systems compare meaning, not only exact words. For example, "refund policy" and "return rules" can be close in vector space. The tradeoff is that semantic similarity can retrieve related but incorrect content. In production, I would evaluate retrieval quality, use metadata filters, and log failed queries.
```

## System Design Pattern

For AI system design questions, explain:

1. User flow.
2. API design.
3. Data storage.
4. Model or retrieval flow.
5. Latency and cost controls.
6. Security and abuse prevention.
7. Monitoring.
8. Failure handling.

## Common Interview Traps

| Trap | Why It Is Weak | Better Answer |
| --- | --- | --- |
| "RAG removes hallucinations" | RAG only reduces some hallucinations | RAG improves grounding, but retrieval and generation can still fail |
| "Temperature controls accuracy" | Temperature controls randomness, not truth | Use retrieval, evaluation, and constraints for factuality |
| "Agents are always better" | Agents add cost, latency, and failure modes | Use agents when the task needs tools, state, or multi-step decisions |
| "Vector DB is always required" | Small systems may not need it | Use vector DBs when semantic retrieval, scale, or metadata filtering matter |
| "Fine-tuning teaches facts" | Fine-tuning is not ideal as a knowledge database | Use RAG for changing knowledge, fine-tuning for behavior or format |

