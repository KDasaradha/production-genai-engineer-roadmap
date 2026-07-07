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
| `devops-notes.md` | DevOps, DevSecOps, deployment options, CI/CD, GitOps, supply chain security, Kubernetes, observability, platform engineering, compliance, and portfolio project notes |
| `archive/duplicate-sources/AI-RAG-Reasearch-notes/rag-chatbots/*.md` | RAG variant prioritization and production RAG learning roadmap |
| `archive/duplicate-sources/AI-RAG-Reasearch-notes/UK COuncil PLanning Portal Production-Grade RAG.md`, `archive/duplicate-sources/AI-RAG-Reasearch-notes/FInalized Architecture.md` | UK Council Planning Portal AI Platform architecture, RAG design, rule engine, multi-tenancy, auditability, and production concerns |
| `archive/duplicate-sources/AI-RAG-Reasearch-notes/AI-Engineer-Roadmap.md` | Mostly overlaps with the master roadmap, project catalog, RAG, agents, LLM engineering, and deployment sections |
| `archive/duplicate-sources/AI-RAG-Reasearch-notes/Devsecops-llm-notes.md` | Mostly overlaps with DevOps and DevSecOps notes, especially CI/CD, OIDC, SBOM, scanning, signing, Kubernetes, and observability |

## Added Consolidation Files

The following files were created to capture cross-source material that did not exist cleanly as single-topic curated notes:

- [Learning-Roadmap.md](../00-Master-Roadmap/Learning-Roadmap.md)
- [Learning-Index.md](../00-Master-Roadmap/Learning-Index.md)
- [Progress-Tracker.md](../00-Master-Roadmap/Progress-Tracker.md)
- [Bootcamp-Schedule.md](../00-Master-Roadmap/Bootcamp-Schedule.md)
- [concurrency-models.md](../01-Python/concurrency-models.md)
- [postgresql-advanced-patterns.md](../03-PostgreSQL/postgresql-advanced-patterns.md)
- [redis-advanced-patterns.md](../04-Redis/redis-advanced-patterns.md)
- [backend-and-ai-scenarios.md](../05-System-Design/backend-and-ai-scenarios.md)
- [system-design-interview-questions.md](../05-System-Design/system-design-interview-questions.md)
- [project-catalog.md](../12-Projects/project-catalog.md)
- [ai-text-summarizer.md](../12-Projects/ai-text-summarizer.md)
- [multi-provider-ai-playground.md](../12-Projects/multi-provider-ai-playground.md)
- [semantic-search-engine.md](../12-Projects/semantic-search-engine.md)
- [knowledge-assistant.md](../12-Projects/knowledge-assistant.md)
- [resume-analyzer.md](../12-Projects/resume-analyzer.md)
- [contract-analyzer.md](../12-Projects/contract-analyzer.md)
- [prompt-evaluation-system.md](../12-Projects/prompt-evaluation-system.md)
- [ai-streaming-chat-api.md](../12-Projects/ai-streaming-chat-api.md)
- [ai-workflow-automation.md](../12-Projects/ai-workflow-automation.md)
- [saas-level-ai-product.md](../12-Projects/saas-level-ai-product.md)
- [coding-assistant.md](../12-Projects/coding-assistant.md)
- [enterprise-rag-platform.md](../12-Projects/enterprise-rag-platform.md)
- [multi-agent-research-system.md](../12-Projects/multi-agent-research-system.md)
- [offline-local-ai-system.md](../12-Projects/offline-local-ai-system.md)
- [fine-tuned-domain-assistant.md](../12-Projects/fine-tuned-domain-assistant.md)
- [ai-security-gateway.md](../12-Projects/ai-security-gateway.md)
- [production-ai-platform.md](../12-Projects/production-ai-platform.md)
- [topic-question-banks.md](../13-Interview-Preparation/topic-question-banks.md)
- [python-interview-questions.md](../13-Interview-Preparation/python-interview-questions.md)
- [fastapi-interview-questions.md](../13-Interview-Preparation/fastapi-interview-questions.md)
- [postgresql-interview-questions.md](../13-Interview-Preparation/postgresql-interview-questions.md)
- [redis-interview-questions.md](../13-Interview-Preparation/redis-interview-questions.md)
- [genai-interview-questions.md](../13-Interview-Preparation/genai-interview-questions.md)
- [rag-interview-questions.md](../13-Interview-Preparation/rag-interview-questions.md)
- [agentic-ai-interview-questions.md](../13-Interview-Preparation/agentic-ai-interview-questions.md)
- [system-design-interview-questions.md](../13-Interview-Preparation/system-design-interview-questions.md)
- [rag-variants-and-production-roadmap.md](../09-RAG/rag-variants-and-production-roadmap.md)
- [career-roadmap-and-study-plan.md](../14-Career/career-roadmap-and-study-plan.md)
- [job-first-track.md](../14-Career/job-first-track.md)
- [Note-Templates-and-Prompts.md](Note-Templates-and-Prompts.md)
- [15-DevOps-DevSecOps/README.md](../15-DevOps-DevSecOps/README.md)
- [15-DevOps-DevSecOps/01-platform-engineer-roadmap.md](../15-DevOps-DevSecOps/01-platform-engineer-roadmap.md)
- [15-DevOps-DevSecOps/02-deployment-fundamentals.md](../15-DevOps-DevSecOps/02-deployment-fundamentals.md)
- [15-DevOps-DevSecOps/03-cloud-registries-and-tooling.md](../15-DevOps-DevSecOps/03-cloud-registries-and-tooling.md)
- [15-DevOps-DevSecOps/04-cicd-github-actions.md](../15-DevOps-DevSecOps/04-cicd-github-actions.md)
- [15-DevOps-DevSecOps/05-devsecops-supply-chain-security.md](../15-DevOps-DevSecOps/05-devsecops-supply-chain-security.md)
- [15-DevOps-DevSecOps/06-kubernetes-helm-gitops.md](../15-DevOps-DevSecOps/06-kubernetes-helm-gitops.md)
- [15-DevOps-DevSecOps/07-production-platform-engineering.md](../15-DevOps-DevSecOps/07-production-platform-engineering.md)
- [15-DevOps-DevSecOps/08-observability-sre-compliance.md](../15-DevOps-DevSecOps/08-observability-sre-compliance.md)
- [15-DevOps-DevSecOps/09-deployment-strategies.md](../15-DevOps-DevSecOps/09-deployment-strategies.md)
- [15-DevOps-DevSecOps/10-portfolio-projects-and-checklists.md](../15-DevOps-DevSecOps/10-portfolio-projects-and-checklists.md)
- [uk-council-planning-portal-ai-platform.md](../12-Projects/uk-council-planning-portal-ai-platform.md)

## Related Topics

- [Repository-Overview.md](Repository-Overview.md)
- [Original-Notes-Index.md](Original-Notes-Index.md)
