# Source Map

## Purpose

This file records how the new `knowledge-base/` was assembled without modifying the original sources.

## Consolidation Rules Applied

- Original files were treated as read-only.
- Curated notes in `notes/` were copied into the new categorized structure where they already represented clean single-topic material.
- Raw and Jupyter notes were reviewed for unique material and consolidated into additional knowledge-base files only where they added coverage.

## Source Coverage

| Source Group | Primary Use in Knowledge Base |
| --- | --- |
| `notes/phase-00-backend-foundation/*.md` | Core Python, FastAPI, PostgreSQL, Redis, Docker, and backend architecture topic files |
| `notes/phase-01-llm-foundations/*.md` | Generative AI foundation topic files |
| `notes/phase-02-prompt-engineering/*.md` | Prompt engineering topic files |
| `notes/phase-03-embeddings-vector-dbs/*.md` | Retrieval foundation topic files |
| `notes/phase-04-production-rag/*.md` | Production RAG topic files |
| `notes/phase-05-ai-agents/*.md` | Agentic AI topic files |
| `notes/phase-06-ai-app-development/*.md` | Software architecture and AI application design files |
| `notes/phase-07-fine-tuning-local-models/*.md` | LLM engineering topic files |
| `notes/phase-08-production-deployment/*.md` | Deployment, cloud, observability, and cost files |
| `notes/interview-prep.md` | Master interview guidance |
| `notes/portfolio-projects.md` | Project ladder and flagship project inputs |
| `raw-notes/chat-001.md`, `raw-notes/chat-002.md` | Knowledge-base structure, learning workflow, and study-plan framing |
| `raw-notes/chat-003.md`, `raw-notes/chat-006.md` | Backend/Python interview-oriented reinforcement already represented in curated phase-00 notes plus the concurrency summary |
| `raw-notes/chat-004.md` | Advanced PostgreSQL correctness topics |
| `raw-notes/Master Learning Prompt.md`, `raw-notes/Recommended Master Learning Prompt.md`, `Agents.md` | Teaching and note-structure guidance preserved as reference material |
| `juppyter-notes/notes/redis-notes.md` | Redis advanced patterns and production usage |
| `juppyter-notes/notebooks/phase1-md/*.md` | Backend day-by-day notes already incorporated into curated phase-00 files |
| `juppyter-notes/notebooks/Python Backend Master Notes.md` | Python concurrency quick-reference topics |
| `raw-notes/chat-005.md`, `raw-notes/roadmap.md`, `Roadmap.md` | Career and roadmap framing |

## Added Consolidation Files

The following files were created to capture cross-source material that did not exist cleanly as single-topic curated notes:

- [Learning-Roadmap.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/00-Master-Roadmap/Learning-Roadmap.md)
- [Learning-Index.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/00-Master-Roadmap/Learning-Index.md)
- [Progress-Tracker.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/00-Master-Roadmap/Progress-Tracker.md)
- [Bootcamp-Schedule.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/00-Master-Roadmap/Bootcamp-Schedule.md)
- [concurrency-models.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/01-Python/concurrency-models.md)
- [postgresql-advanced-patterns.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/03-PostgreSQL/postgresql-advanced-patterns.md)
- [redis-advanced-patterns.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/04-Redis/redis-advanced-patterns.md)
- [backend-and-ai-scenarios.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/05-System-Design/backend-and-ai-scenarios.md)
- [system-design-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/05-System-Design/system-design-interview-questions.md)
- [project-catalog.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/project-catalog.md)
- [ai-text-summarizer.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/ai-text-summarizer.md)
- [multi-provider-ai-playground.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/multi-provider-ai-playground.md)
- [semantic-search-engine.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/semantic-search-engine.md)
- [knowledge-assistant.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/knowledge-assistant.md)
- [resume-analyzer.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/resume-analyzer.md)
- [contract-analyzer.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/contract-analyzer.md)
- [prompt-evaluation-system.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/prompt-evaluation-system.md)
- [ai-streaming-chat-api.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/ai-streaming-chat-api.md)
- [ai-workflow-automation.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/ai-workflow-automation.md)
- [saas-level-ai-product.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/saas-level-ai-product.md)
- [coding-assistant.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/coding-assistant.md)
- [enterprise-rag-platform.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/enterprise-rag-platform.md)
- [multi-agent-research-system.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/multi-agent-research-system.md)
- [offline-local-ai-system.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/offline-local-ai-system.md)
- [fine-tuned-domain-assistant.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/fine-tuned-domain-assistant.md)
- [ai-security-gateway.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/ai-security-gateway.md)
- [production-ai-platform.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/production-ai-platform.md)
- [topic-question-banks.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/topic-question-banks.md)
- [python-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/python-interview-questions.md)
- [fastapi-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/fastapi-interview-questions.md)
- [postgresql-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/postgresql-interview-questions.md)
- [redis-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/redis-interview-questions.md)
- [genai-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/genai-interview-questions.md)
- [rag-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/rag-interview-questions.md)
- [agentic-ai-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/agentic-ai-interview-questions.md)
- [system-design-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/system-design-interview-questions.md)
- [career-roadmap-and-study-plan.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/14-Career/career-roadmap-and-study-plan.md)
- [job-first-track.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/14-Career/job-first-track.md)
- [Note-Templates-and-Prompts.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/Resources/Note-Templates-and-Prompts.md)

## Related Topics

- [Repository-Overview.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/Resources/Repository-Overview.md)
- [Original-Notes-Index.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/Resources/Original-Notes-Index.md)
