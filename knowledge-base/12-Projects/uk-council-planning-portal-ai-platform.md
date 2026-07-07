# UK Council Planning Portal AI Platform

This page consolidates the unique production project ideas from `archive/duplicate-sources/AI-RAG-Reasearch-notes/UK COuncil PLanning Portal Production-Grade RAG.md` and `archive/duplicate-sources/AI-RAG-Reasearch-notes/FInalized Architecture.md`.

## Problem Statement

Build an AI assistant for UK council planning services that helps residents, council staff, and case workers understand planning guidance, eligibility questions, required documents, and council-specific rules.

The system should not behave like a generic chatbot. It should ground answers in council-specific guidance, cite sources, apply rule logic, enforce tenant isolation, and route uncertain cases to humans.

## Use When

| Suitable Scenario | Why |
| --- | --- |
| Council-specific planning guidance | Rules vary by council, service, and document version |
| User needs explainable answers | Citations and rule explanations build trust |
| Eligibility-style workflows | Rules and forms need deterministic structure |
| Multi-tenant public-sector platform | Each council needs isolated data and permissions |

## Avoid When

| Wrong Use Case | Better Alternative |
| --- | --- |
| Simple static FAQ | Search page or curated FAQ |
| No authoritative documents | Do not pretend the AI knows the answer |
| Legal final decision-making | Human officer review and official process |
| Unclear tenant boundaries | Fix access control before adding AI |

## Architecture

```text
User
  |
  v
Web / Mobile UI
  |
  v
FastAPI Gateway
  |
  +--> Auth / Tenant Resolver
  |
  +--> Planning Rule Engine
  |
  +--> RAG Query Service
  |      |
  |      +--> Query Rewrite
  |      +--> Metadata Filter
  |      +--> Hybrid Retrieval
  |      +--> Reranker
  |      +--> Citation Builder
  |
  +--> LLM Gateway
  |
  +--> Audit Log / Feedback

Ingestion Worker
  |
  +--> Council Documents
  +--> Planning Rules
  +--> OCR / Parsing
  +--> Chunking
  +--> Embeddings
  +--> Vector DB
  +--> PostgreSQL Metadata
```

## Core Components

| Component | Responsibility |
| --- | --- |
| FastAPI Gateway | API endpoints, auth, rate limits, request validation |
| Tenant Resolver | Maps user/session to council, role, permissions |
| Rule Engine | Handles deterministic planning logic and eligibility checks |
| RAG Service | Retrieves grounded guidance and citations |
| LLM Gateway | Centralizes model calls, prompt templates, retries, cost tracking |
| PostgreSQL | Councils, services, rules, documents, versions, users, audit logs |
| Vector Database | Searchable chunks with council/service/version metadata |
| Redis | Sessions, rate limits, queues, short-lived cache |
| Worker Queue | Ingestion, parsing, embedding, reindexing |
| Admin Portal | Upload documents, manage rules, review feedback |

## Data Model

| Table | Purpose |
| --- | --- |
| `councils` | Tenant identity and council metadata |
| `services` | Planning services, forms, categories |
| `rules` | Deterministic eligibility and validation rules |
| `documents` | Source documents and versions |
| `document_chunks` | Chunk metadata, source references, embedding IDs |
| `queries` | User questions, retrieved sources, answer metadata |
| `feedback` | User corrections, quality signals, human review |
| `audit_logs` | Security and compliance trail |

## RAG Design

| Step | Production Detail |
| --- | --- |
| Ingestion | Parse PDFs, web pages, council guidance, forms, and policy files |
| Chunking | Preserve section headings, service names, council names, and document versions |
| Metadata | Store `council_id`, `service_id`, `document_type`, `version`, `effective_date`, `source_url` |
| Retrieval | Use hybrid search with metadata filters before reranking |
| Reranking | Prefer council-specific and current documents |
| Citations | Return source title, section, version, and URL |
| Guardrails | Refuse unsupported legal conclusions; show uncertainty |

## Why Hybrid RAG

Planning guidance contains both semantic questions and exact terms such as service names, document codes, building types, locations, and council-specific phrases. Pure vector search can miss exact matches. Pure keyword search can miss meaning. Hybrid RAG is the safer default.

## Production Considerations

| Area | Requirement |
| --- | --- |
| Security | Tenant isolation, role-based retrieval, prompt injection protection |
| Compliance | Audit logs, source citations, retention policies |
| Evaluation | Golden question set per council, citation accuracy, retrieval recall |
| Observability | Trace ingestion, retrieval, reranking, LLM calls, refusals |
| Cost | Cache frequent answers, batch embeddings, route models by difficulty |
| Failure Handling | Fall back to search results or human review when confidence is low |
| Versioning | Keep old document versions for auditability |

## API Sketch

```text
POST /api/v1/planning/query
POST /api/v1/planning/eligibility-check
GET  /api/v1/councils/{council_id}/services
POST /api/v1/admin/documents
POST /api/v1/admin/rules
GET  /api/v1/admin/evaluations
POST /api/v1/feedback
```

## Project Milestones

| Phase | Build |
| --- | --- |
| 1 | Council, service, document, and rule data model |
| 2 | Document ingestion and chunk metadata pipeline |
| 3 | Hybrid RAG query API with citations |
| 4 | Rule engine for eligibility-style questions |
| 5 | Admin portal for documents, rules, and feedback |
| 6 | Evaluation dashboard and audit reports |
| 7 | Production deployment with monitoring and security checks |

## Interview Answer

I would design this as a multi-tenant AI platform, not a generic chatbot. The core is PostgreSQL for councils, services, rules, documents, versions, and audit logs; a vector database for council-filtered document chunks; Redis for rate limits and caching; FastAPI for APIs; and a hybrid RAG pipeline with metadata filtering, reranking, citations, evaluation, and human fallback. Deterministic planning rules should live in a rule engine, while RAG explains guidance and cites sources.

Are you ready for the next section?
