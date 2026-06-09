# Learning Roadmap

## Goal

Move from Python backend developer to production-ready GenAI engineer in an order that matches how real AI systems are built.

## Learning Sequence

| Stage | Focus Area | Why It Comes Here | Primary Output |
| --- | --- | --- | --- |
| 0 | Orientation | You need a clear path before studying deeply | Schedule, tracker, and execution rhythm |
| 1 | Python backend foundations | Async apps, streaming APIs, and AI services depend on strong Python fundamentals | Async, generators, decorators, context managers, typing, and errors |
| 2 | FastAPI | AI products need reliable APIs, validation, dependency injection, and lifecycle control | Production-ready API services |
| 3 | PostgreSQL and Redis | RAG and agent systems need durable storage, caching, sessions, queues, and rate limits | Reliable data and performance layer |
| 4 | Generative AI foundations | Prompting, RAG, and agents are weak without tokens, embeddings, context, and model behavior | Correct LLM mental model |
| 5 | Prompt engineering | Prompt design is the fastest way to improve output quality before building larger systems | Reliable prompts and structured outputs |
| 6 | Retrieval and RAG | Grounding is the core production pattern for enterprise AI systems | Document Q&A, search, citations, and retrieval evaluation |
| 7 | Agentic AI | Agents add tools, planning, state, and orchestration on top of LLM/RAG foundations | Tool-using multi-step workflows |
| 8 | Software architecture | Demos become products only when logging, security, deployment, UX, and observability exist | Production AI application architecture |
| 9 | LLM engineering | Fine-tuning, quantization, and local serving trade off cost, control, and latency | Model customization and serving skills |
| 10 | System design | Interviews test tradeoffs across the whole system | Design-ready AI backend thinking |
| 11 | Projects and career | Hiring needs proof of skill, not only notes | Portfolio, interview practice, and job plan |
| 12 | DevOps and DevSecOps | Production backend roles expect secure delivery, cloud deployment, observability, and operations | Source-code-to-production platform skills |

## Recommended Study Order

1. [01-Python](../01-Python/README.md)
2. [02-FastAPI](../02-FastAPI/README.md)
3. [03-PostgreSQL](../03-PostgreSQL/README.md)
4. [04-Redis](../04-Redis/README.md)
5. [08-Generative-AI](../08-Generative-AI/README.md)
6. [07-Prompt-Engineering](../07-Prompt-Engineering/README.md)
7. [09-RAG](../09-RAG/README.md)
8. [10-Agentic-AI](../10-Agentic-AI/README.md)
9. [06-Software-Architecture](../06-Software-Architecture/README.md)
10. [11-LLM-Engineering](../11-LLM-Engineering/README.md)
11. [05-System-Design](../05-System-Design/README.md)
12. [12-Projects](../12-Projects/README.md)
13. [13-Interview-Preparation](../13-Interview-Preparation/README.md)
14. [14-Career](../14-Career/README.md)
15. [15-DevOps-DevSecOps](../15-DevOps-DevSecOps/README.md)

## Fastest Job-Ready Track

If speed matters more than completeness, prioritize these topics:

| Step | Topics | Build Evidence |
| --- | --- | --- |
| 1 | AsyncIO, generators, error handling, FastAPI, Pydantic | Streaming API with validation and error handling |
| 2 | PostgreSQL performance, Redis caching | API with persistence, cache, and rate limits |
| 3 | Tokens, embeddings, cosine similarity, hallucinations | Semantic search demo |
| 4 | Prompt types, structured outputs, prompt security | Prompt evaluation mini system |
| 5 | Chunking, retrieval, vector databases, RAG pipeline | Knowledge assistant with citations |
| 6 | Tool calling, state, memory, orchestration | Multi-step agent workflow |
| 7 | Observability, cloud architecture, CI/CD, cost controls | Production AI platform design |
| 8 | Docker, GitHub Actions, AWS, ECS/EKS, ArgoCD, DevSecOps | Secure deployment portfolio project |

## Weekly Routine

| Day | Focus |
| --- | --- |
| Monday | Learn one topic deeply and summarize it |
| Tuesday | Rebuild the Python example from memory |
| Wednesday | Add a FastAPI, database, or Redis version |
| Thursday | Study failures, tradeoffs, and production concerns |
| Friday | Practice interview answers and design tradeoffs |
| Saturday | Build or extend a project |
| Sunday | Revise weak areas and update the progress tracker |

## Milestones

| Milestone | Evidence |
| --- | --- |
| Backend-ready | You can explain async vs sync, build FastAPI APIs, and use PostgreSQL and Redis safely |
| LLM-ready | You can explain tokens, embeddings, context windows, temperature, and hallucinations clearly |
| RAG-ready | You can build ingestion, retrieval, reranking, and citation flows |
| Agent-ready | You can justify when agents are useful and control tools, loops, state, and memory |
| Production-ready | You can discuss logging, monitoring, scaling, security, failure modes, and cost |
| Deployment-ready | You can containerize, build CI/CD, deploy to cloud, secure artifacts, observe production, and handle rollback |

## Related Files

- [Learning-Index.md](Learning-Index.md)
- [Bootcamp-Schedule.md](Bootcamp-Schedule.md)
- [Progress-Tracker.md](Progress-Tracker.md)
- [career-roadmap-and-study-plan.md](../14-Career/career-roadmap-and-study-plan.md)
- [15-DevOps-DevSecOps](../15-DevOps-DevSecOps/README.md)
