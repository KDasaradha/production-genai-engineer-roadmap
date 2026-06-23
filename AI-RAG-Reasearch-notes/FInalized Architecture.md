# Finalized Architecture

## UK Council Planning Portal AI Assistant (Production Grade - 2026)

This architecture is optimized for:

* Your background: Python Backend Developer
* AI Engineer roadmap
* Open-source first
* Production-grade RAG
* Multi-council support
* Multi-service support
* Future Agent support
* FastAPI ecosystem
* Maintainability
* Portfolio quality

---

# 1. System Overview

```text
┌──────────────────────────────┐
│         Next.js UI           │
│                              │
│ Chat Assistant               │
│ Planning Portal              │
│ Forms                         │
│ Applications                  │
└──────────────┬───────────────┘
               │
               │ HTTPS
               │
┌──────────────▼───────────────┐
│         API Gateway          │
│          FastAPI             │
└──────────────┬───────────────┘
               │
     ┌─────────┼──────────┐
     │         │          │
     ▼         ▼          ▼

 Planning   AI Chat     Auth
 Services   Service     Service

     │         │
     │         ▼

     │   AI Orchestrator
     │
     │────────────┬────────────┐
                  │            │
                  ▼            ▼

           Rule Engine      RAG Engine

                  │            │
                  ▼            ▼

              MongoDB      Qdrant

                               │
                               ▼

                         OpenSearch

                               │
                               ▼

                           Reranker

                               │
                               ▼

                             LLM
```

---

# 2. Technology Stack

## Frontend

| Component     | Technology      |
| ------------- | --------------- |
| UI            | Next.js 15      |
| Language      | TypeScript      |
| Styling       | TailwindCSS     |
| State         | Zustand         |
| Data Fetching | TanStack Query  |
| Forms         | React Hook Form |
| Validation    | Zod             |

---

## Backend

| Component            | Technology  |
| -------------------- | ----------- |
| API                  | FastAPI     |
| Language             | Python 3.13 |
| Validation           | Pydantic v2 |
| ORM                  | Beanie ODM  |
| Auth                 | JWT RS256   |
| Dependency Injection | FastAPI DI  |
| Async Runtime        | AsyncIO     |

---

## Database

### Primary Database

```text
MongoDB
```

Stores:

* Users
* Councils
* Services
* Forms
* Applications
* Rules
* Chat History
* Audit Logs
* Feedback

---

## Vector Database

```text
Qdrant
```

Stores:

* Embeddings
* Chunks
* Metadata

---

## Search Engine

```text
OpenSearch
```

Stores:

* BM25 indexes
* Keyword indexes

---

## Cache

```text
Redis
```

Stores:

* Sessions
* Retrieval cache
* LLM cache
* Rate limiting

---

# 3. AI Stack

## LLM

### Development

```text
Ollama
Qwen3 14B
```

---

### Production

```text
vLLM
Qwen3 32B
```

---

## Embeddings

```text
BAAI/bge-m3
```

Reason:

* Open Source
* Multilingual
* Excellent retrieval

---

## Reranker

```text
BAAI/bge-reranker-v2-m3
```

Reason:

* Significant retrieval improvement

---

# 4. AI Components

---

## Component 1

### Rule Engine

Purpose:

Explain form behavior.

Example:

```text
Why am I seeing Question 17?
```

Rule Engine responds.

---

### Data

```json
{
  "rule_id":"RULE_17",
  "service":"planning_permission",
  "condition":"listed_building=true",
  "reason":"Listed buildings require additional information"
}
```

---

## Component 2

### Hybrid RAG Engine

Purpose:

Knowledge retrieval.

Sources:

* PDFs
* Policies
* FAQs
* Guidance
* Service documentation

---

Pipeline:

```text
Question

↓

Query Rewriter

↓

BM25 Search

+

Vector Search

↓

RRF Fusion

↓

Reranker

↓

Context Builder

↓

LLM
```

---

## Component 3

### Tool Engine

Purpose:

Access application data.

Examples:

```text
Get Application Status

Get Council Details

Get Service Details

Get User Progress
```

---

## Component 4

### Chat Engine

Purpose:

Conversation management.

Stores:

```text
Session
Messages
Sources
Feedback
```

---

# 5. MongoDB Collections

## Users

```text
users
```

---

## Councils

```text
councils
```

---

## Services

```text
services
```

---

## Forms

```text
forms
```

---

## Questions

```text
questions
```

---

## Rules

```text
rules
```

---

## Applications

```text
applications
```

---

## Chat Sessions

```text
chat_sessions
```

---

## Chat Messages

```text
chat_messages
```

---

## Audit Logs

```text
audit_logs
```

---

## Feedback

```text
feedback
```

---

# 6. Qdrant Collections

Only one collection initially.

---

## planning_knowledge

Stores:

```text
Council Guidance

Policies

FAQs

Planning Rules

Services

Documents
```

---

Payload Example

```json
{
  "document_id":"DOC001",
  "document_type":"guidance",
  "council_id":"BRISTOL",
  "service_id":"planning_permission",
  "chunk_id":"chunk_01",
  "version":"1.0"
}
```

---

# 7. OpenSearch Indexes

---

## planning_documents

Stores:

```text
Full text search
```

---

## faq_documents

Stores:

```text
FAQ search
```

---

# 8. Metadata Strategy

Every chunk must contain:

```json
{
  "document_id":"",
  "document_name":"",
  "document_type":"",
  "council_id":"",
  "service_id":"",
  "form_id":"",
  "version":"",
  "created_at":""
}
```

---

# 9. Ingestion Pipeline

```text
Upload

↓

Validation

↓

Document Extraction

↓

Cleaning

↓

Chunking

↓

Embedding

↓

Qdrant

↓

OpenSearch
```

---

# 10. Chunking Strategy

Use:

```text
Parent Child Chunking
```

Parent:

```text
Planning Permission Guidance
```

Children:

```text
500 token chunks
```

Recommended:

```text
500
overlap 100
```

---

# 11. Security Architecture

## Authentication

```text
JWT RS256
```

---

## Authorization

RBAC

Roles:

```text
Customer
Consultant
Employee
Admin
```

---

## Retrieval Security

Mandatory filters:

```text
Council

Service

Role
```

---

Example:

```python
filters = {
   "council_id": council_id
}
```

---

# 12. Observability

## Metrics

Track:

```text
Latency
Token Usage
Retrieval Time
Cache Hit Rate
Errors
```

---

## Stack

```text
Prometheus

Grafana

OpenTelemetry
```

---

# 13. Background Processing

## Celery

Purpose:

```text
Document Processing

Embeddings

Reindexing

Long Running Tasks
```

---

## Redis

Broker:

```text
Redis
```

---

# 14. Deployment

```text
NGINX

↓

Next.js

↓

FastAPI

↓

MongoDB

↓

Redis

↓

Qdrant

↓

OpenSearch

↓

Ollama/vLLM
```

Dockerized.

---

# 15. Folder Structure (Final)

```text
ai-platform/

├── app/

│   ├── api/
│   │   ├── v1/
│   │   ├── dependencies/
│   │   └── middleware/

│   ├── core/
│   │   ├── config/
│   │   ├── security/
│   │   ├── logging/
│   │   └── constants/

│   ├── modules/

│   │   ├── auth/

│   │   ├── users/

│   │   ├── councils/

│   │   ├── services/

│   │   ├── forms/

│   │   ├── applications/

│   │   ├── chat/

│   │   ├── rules/

│   │   ├── rag/
│   │   │   ├── ingestion/
│   │   │   ├── chunking/
│   │   │   ├── embeddings/
│   │   │   ├── retrieval/
│   │   │   ├── reranking/
│   │   │   ├── prompts/
│   │   │   └── services/

│   │   ├── llm/
│   │   │   ├── providers/
│   │   │   └── gateway/

│   │   ├── evaluation/

│   │   └── audit/

│   ├── db/

│   ├── workers/

│   └── tests/

├── alembic/ (optional if SQL later)

├── docker/

├── scripts/

└── docs/
```

---

# Phase Roadmap

### Phase 1 (Must Build)

* FastAPI
* MongoDB
* Redis
* Qdrant
* Ollama
* Chat UI
* Basic RAG

### Phase 2

* OpenSearch
* Hybrid Search
* Reranker
* Citations

### Phase 3

* Rule Engine
* Council Filtering
* Service Filtering

### Phase 4

* Tool Calling
* Application Assistance
* Form Assistance

### Phase 5

* LangGraph Workflows
* Multi-step Planning Assistant

### Phase 6

* Multi-agent Architecture
* Knowledge Graph
* GraphRAG (if actually needed)

This is the architecture I would choose for **your specific situation**: Python-focused, open-source-first, production-oriented, and realistic enough to be built incrementally while also serving as a strong AI Engineer portfolio project.

---

Yes. The architecture we finalized is about **80-85% complete**.

Most developers stop here and immediately start coding.

That is usually where projects become messy after a few months.

Before writing code, there are still some critical design decisions you should finalize.

---

# 1. Knowledge Modeling (Most Important)

This is more important than choosing Qdrant, Ollama, or FastAPI.

You need to define:

```text
Council
   |
Service
   |
Form
   |
Section
   |
Question
   |
Rule
   |
Document
```

For example:

```text
Bristol Council

    ↓

Householder Planning Permission

    ↓

Eligibility Form

    ↓

Question 22

    ↓

Listed Building Rule
```

This becomes your AI Knowledge Graph (not GraphRAG).

Without this model:

```text
RAG becomes confusing
Metadata becomes inconsistent
Filtering becomes difficult
```

---

# 2. Multi-Tenant Design

This is extremely important.

You have:

```text
Council A

Council B

Council C
```

Questions:

```text
Can councils share documents?

Can councils override services?

Can councils override forms?

Can councils override rules?
```

Example:

```text
Global Planning Service

     ↓

Council Specific Override
```

Decide this before coding.

---

# 3. Document Versioning

Most RAG tutorials ignore this.

Example:

```text
Planning Guidance v1

Planning Guidance v2
```

Questions:

```text
Delete old embeddings?

Keep both?

Support rollback?

Which version should AI answer from?
```

I strongly recommend:

```text
Version everything
```

Documents

Chunks

Embeddings

Rules

Prompts

---

# 4. Source of Truth

You must define:

## Who owns the data?

Example:

```text
MongoDB
    ↓
Source of Truth

Qdrant
    ↓
Search Copy

OpenSearch
    ↓
Search Copy
```

Never edit data directly in:

```text
Qdrant
OpenSearch
```

---

# 5. Prompt Management

Most AI projects become unmaintainable here.

Do NOT do:

```python
PROMPT = """
...
"""
```

everywhere.

Create:

```text
Prompt Registry
```

Examples:

```text
rag_answer_prompt

rule_explanation_prompt

document_summary_prompt

council_guidance_prompt
```

Store versions.

---

# 6. Evaluation Framework

Most important production feature.

Before launch:

Create evaluation datasets.

Example:

```text
Question

Expected Answer

Expected Source
```

Then test:

```text
RAG Retrieval

Reranking

LLM Output
```

Use:

* RAGAS
* DeepEval

---

# 7. Human Feedback Loop

Add:

```text
👍 Helpful

👎 Not Helpful
```

to every answer.

Store:

```text
Question
Answer
Sources
Feedback
```

This becomes gold for improving the system.

---

# 8. AI Cost Strategy

Even with open source.

You still pay:

```text
CPU

RAM

GPU

Storage

Embedding Generation
```

Track:

```text
Prompt Tokens

Completion Tokens

Latency

Cache Hits
```

from day one.

---

# 9. Failure Strategy

Most projects fail here.

What happens if:

```text
Qdrant Down?

OpenSearch Down?

Redis Down?

LLM Down?

Embedding Service Down?
```

Example:

```text
Qdrant unavailable

→ Fallback BM25 Search
```

Design this now.

---

# 10. Security for RAG

Very few tutorials discuss this.

Users can try:

```text
Ignore instructions

Show all documents

Reveal internal guidance
```

You need:

### Prompt Injection Protection

### Role Based Retrieval

### Metadata Filtering

### Output Validation

---

# 11. Future Agent Architecture

Don't build agents yet.

But leave space.

Current:

```text
User

 ↓

RAG
```

Future:

```text
User

 ↓

Router

 ↓

Agent

 ↓

Tools

 ↓

RAG
```

Design modules so this can be added later.

---

# 12. Admin Portal (Often Forgotten)

You will eventually need:

```text
Upload Documents

Manage Councils

Manage Services

Manage Rules

Manage Prompts

View Feedback

View Analytics
```

Design for this now.

---

# 13. Production DevOps

For a real portfolio-grade project, include:

```text
Docker

Docker Compose

GitHub Actions

Pytest

Pre-commit

Ruff

Mypy

Bandit

Trivy

Dependabot
```

This is one of the biggest differences between:

```text
AI Demo Project

and

Production AI Platform
```

---

# 14. Things I Would Add to Your Final Architecture

### AI Gateway

```text
Client

 ↓

AI Gateway

 ↓

Services
```

Handles:

* Authentication
* Rate Limiting
* Logging
* Monitoring

---

### Prompt Registry

```text
MongoDB
```

Collection:

```text
prompt_templates
```

---

### Evaluation Module

```text
modules/evaluation
```

---

### Analytics Module

```text
modules/analytics
```

---

### Feedback Module

```text
modules/feedback
```

---

# What You Need Before Coding

I would not start coding until these 5 documents are finalized:

## 1. HLD ✅

(Already done)

## 2. LLD

(Not done yet)

Contains:

```text
Mongo Schemas
Qdrant Collections
API Contracts
Services
Sequence Diagrams
```

---

## 3. RAG Design Document

Contains:

```text
Chunking Strategy
Metadata Schema
Retrieval Flow
Reranking Flow
Citation Strategy
```

---

## 4. Rule Engine Design

Contains:

```text
Rule Schema
Rule Evaluation Flow
Question Explanation Flow
```

---

## 5. Implementation Roadmap

Contains:

```text
Phase 1
Phase 2
Phase 3
...
Phase N
```

with exact tasks.

If you complete those five documents first, Codex/Copilot can generate a very large percentage of the implementation while you stay in control of the architecture rather than constantly redesigning it mid-project.

---

From your list:

| Item                                     | Status     |
| ---------------------------------------- | ---------- |
| Finalize Architecture                    | ✅ Done     |
| Finalize Development Documentation (HLD) | ✅ Done     |
| Finalize Folder Structure                | ✅ Done     |
| Finalize Database Design                 | ❌ Not Done |
| Finalize RAG Design                      | ❌ Not Done |
| Create Master Codex Prompt               | ❌ Not Done |
| Let Codex Generate Module-by-Module      | Future     |
| Review and Refine                        | Future     |

So the **3 major missing artifacts** are:

---

# 1. Database Design (Production)

This is the most important missing piece.

You need complete schemas for:

```text
MongoDB Collections

users
roles
councils
services
forms
questions
rules
documents
applications
chat_sessions
chat_messages
feedback
audit_logs
prompt_templates
```

Need definitions for:

* fields
* indexes
* unique constraints
* relationships
* document structure
* versioning strategy

---

## Example

### councils

```json
{
  "_id": ObjectId,

  "council_code": "BRISTOL",

  "name": "Bristol Council",

  "status": "active",

  "contact_email": "",

  "website": "",

  "created_at": "",

  "updated_at": ""
}
```

Indexes:

```text
council_code UNIQUE
name
status
```

---

### services

```json
{
  "_id": ObjectId,

  "service_code": "HOUSEHOLDER",

  "name": "Householder Planning Permission",

  "council_id": "",

  "description": "",

  "status": "active"
}
```

---

### rules

```json
{
  "_id": ObjectId,

  "rule_code": "RULE_17",

  "service_id": "",

  "question_id": "",

  "condition": {
      "listed_building": true
  },

  "reason": "Listed building requires assessment",

  "version": 1,

  "active": true
}
```

---

### documents

Source of Truth.

```json
{
  "_id": ObjectId,

  "document_code": "DOC001",

  "title": "",

  "council_id": "",

  "service_id": "",

  "version": 1,

  "file_path": "",

  "status": "active",

  "uploaded_at": ""
}
```

---

# 2. Production RAG Design

This is currently the biggest missing document.

---

## Knowledge Sources

```text
Council Guidance
Policies
FAQs
Planning Rules
Service Documentation
Internal SOPs
```

---

## Ingestion Flow

```text
Document Upload

↓

Document Validation

↓

Text Extraction

↓

Cleaning

↓

Chunking

↓

Embeddings

↓

Qdrant

↓

OpenSearch
```

---

## Chunk Strategy

Recommended:

```text
Parent Child Chunking
```

Parent:

```text
Planning Permission Guidance
```

Child:

```text
500 token chunk
```

Configuration:

```yaml
chunk_size: 500

overlap: 100
```

---

## Metadata Schema

Every chunk:

```json
{
  "chunk_id": "",

  "document_id": "",

  "council_id": "",

  "service_id": "",

  "document_type": "",

  "version": "",

  "parent_id": ""
}
```

---

## Retrieval Pipeline

```text
Question

↓

Query Rewrite

↓

Vector Search

(Qdrant)

+

BM25 Search

(OpenSearch)

↓

RRF Fusion

↓

Reranker

↓

Top 5

↓

Context Builder

↓

LLM
```

---

## Citation Strategy

Every answer:

```text
Source:
Document Name

Section

Chunk ID
```

---

## Security Filters

Mandatory:

```python
{
    "council_id": council_id,
    "service_id": service_id
}
```

before retrieval.

---

# 3. Master Codex Prompt

This is what allows Codex/Copilot to generate the system consistently.

---

# Master Prompt

```text
You are a Senior Staff AI Engineer.

Build a production-grade UK Council Planning Portal AI Platform.

Technology:

Frontend:
- Next.js

AI Backend:
- Python 3.13
- FastAPI
- Pydantic v2
- AsyncIO

Database:
- MongoDB
- Beanie ODM

Vector DB:
- Qdrant

Search:
- OpenSearch

Cache:
- Redis

LLM:
- Ollama
- Qwen3

Embeddings:
- BAAI/bge-m3

Reranker:
- BAAI/bge-reranker-v2-m3

Architecture Requirements:

- Clean Architecture
- Domain Driven Design
- Async First
- SOLID Principles
- Production Grade
- Type Hints Everywhere
- Structured Logging
- OpenTelemetry
- JWT RS256
- RBAC
- Rate Limiting
- Dependency Injection

Modules:

- Auth
- Users
- Councils
- Services
- Forms
- Questions
- Rules
- Documents
- Applications
- Chat
- RAG
- LLM
- Feedback
- Audit

RAG Requirements:

- Parent Child Chunking
- Hybrid Search
- Qdrant Vector Search
- OpenSearch BM25
- RRF Fusion
- Reranking
- Citations
- Metadata Filtering

Testing:

- Pytest
- Async Tests
- Integration Tests

DevSecOps:

- Docker
- Docker Compose
- GitHub Actions
- Ruff
- Mypy
- Bandit
- Trivy

Generate code module-by-module.

Do not skip production concerns.

Always generate:
- Folder structure
- Models
- Schemas
- Services
- Repositories
- API Routes
- Tests
- Logging
- Error Handling
```

---

# What Is Still Missing After This?

Only two major documents remain before coding:

## LLD (Low-Level Design)

Contains:

```text
Class Diagrams

Service Design

Repository Design

API Contracts

Sequence Diagrams

Dependency Flow

Request/Response Models
```

---

## Implementation Roadmap

Contains:

```text
Phase 1
Environment Setup

Phase 2
Authentication

Phase 3
Document Ingestion

Phase 4
RAG

Phase 5
Rule Engine

Phase 6
Chat

Phase 7
Observability

Phase 8
DevSecOps

Phase 9
Production Deployment
```

At this point, if you have:

* Architecture
* HLD
* Database Design
* RAG Design
* Master Codex Prompt
* LLD
* Implementation Roadmap

you have essentially the same documentation package that many engineering teams create before starting a serious AI platform.
---
