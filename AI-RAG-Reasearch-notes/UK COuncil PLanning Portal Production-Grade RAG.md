For your **UK Council Planning Portal**, I would **not start with Agentic RAG, Graph RAG, Self-RAG, or CRAG**.

Those are often overkill for the problem you've described.

---

# First Let's Understand Your Use Case

You mentioned:

### Users

* New customers/applicants
* Internal agents/employees
* Possibly council staff

### Questions

Examples:

* "Which planning application should I choose?"
* "Why am I seeing this eligibility question?"
* "Why is this field mandatory?"
* "What documents do I need?"
* "Why did the system ask about conservation areas?"
* "What does this timeout mean?"
* "What happens after submission?"
* "Which service applies to my property?"

### Knowledge Sources

Likely:

* Eligibility rules
* Council guidance documents
* Service definitions
* Application workflows
* Business rules
* FAQs
* Internal SOPs
* Form metadata
* Dynamic eligibility engine outputs

---

# What Type of Problem Is This?

This is primarily:

## Knowledge Retrieval + Workflow Explanation

Not:

* Research Assistant
* Autonomous Agent
* Multi-step Investigation

Therefore:

❌ Agentic RAG first

❌ Graph RAG first

❌ Self-RAG first

---

# What I Would Build

## Advanced Hybrid RAG

Architecture:

```text
User Question
       ↓
Query Rewriting
       ↓

BM25 Search
       +
Vector Search

       ↓
Fusion (RRF)
       ↓

Metadata Filters
       ↓

Reranker
       ↓

Top Context
       ↓

LLM
       ↓

Answer + Citations
```

This is what many enterprise copilots use.

---

# Why Hybrid RAG?

Imagine:

User asks:

```text
Why am I being asked about listed buildings?
```

Keyword Search finds:

```text
listed building
```

exactly.

Semantic Search finds:

```text
heritage protection rules
historic structures
```

related content.

Combined result is much better.

---

# Why Metadata Filtering?

Suppose you have:

```text
Council A
Council B
Council C
```

Same question.

Different answers.

---

Without filtering:

```text
Wrong council data
```

could be retrieved.

---

With metadata:

```python
{
  "council_id": 123,
  "service": "planning_permission"
}
```

Only relevant content is searched.

---

# Your Biggest Requirement

You said:

> why does this eligibility form has different questions

This is not actually a RAG problem.

This is:

## Business Rules Explanation

Example:

```python
IF
property_type == commercial

THEN
show question_17
```

---

Instead of only storing PDFs:

Store rule explanations.

Example:

```json
{
  "rule_id": "RULE_17",
  "condition": "commercial_property",
  "reason": "Commercial developments require parking impact assessment."
}
```

Then RAG can retrieve:

```text
Question 17 is shown because your property was marked as commercial.
```

This becomes much more useful.

---

# Recommended Architecture

## Layer 1

Business Rules Engine

```text
Eligibility Logic
```

Examples:

```python
if is_listed_building:
    show_question_22
```

---

## Layer 2

Knowledge Base

Store:

* Planning guidance
* FAQs
* Service definitions
* Council rules
* SOPs

---

## Layer 3

Hybrid RAG

Retrieve explanations.

---

## Layer 4

LLM

Generate user-friendly responses.

---

# Example Flow

User asks:

```text
Why did Question 22 appear?
```

System:

### Step 1

Read form state.

```json
{
  "listed_building": true
}
```

---

### Step 2

Fetch rule.

```json
{
  "rule": "listed_building_check"
}
```

---

### Step 3

Retrieve guidance.

```text
Listed buildings require additional assessment.
```

---

### Step 4

LLM response.

```text
Question 22 appeared because your property is marked as a listed building.

UK planning regulations require additional information for listed buildings to assess the impact of proposed changes.
```

---

# Should You Use Agents?

Initially:

**No.**

For your first version:

```text
Hybrid RAG
+
Business Rules Lookup
+
Function Calling
```

is enough.

---

# When Agentic RAG Becomes Useful

Later.

Example:

User asks:

```text
Can I build a two-story rear extension?
```

Agent:

1. Check property details
2. Check council
3. Check conservation area
4. Check planning guidance
5. Generate recommendation

Now Agentic RAG makes sense.

---

# My Recommendation for Your Planning Portal

### V1 (Build This)

✅ Hybrid RAG

✅ Metadata Filtering

✅ Reranking

✅ Citations

✅ Business Rules Explanation Layer

✅ Function Calling

---

### V2

✅ Agentic RAG

✅ Form-filling assistance

✅ Eligibility reasoning

✅ Multi-step workflow support

---

### V3

✅ Graph RAG

If you start storing relationships like:

```text
Council
  ↓
Service
  ↓
Form
  ↓
Eligibility Rule
  ↓
Question
```

then Graph RAG becomes valuable.

---

For your specific application, the architecture I would put into production is:

```text
FastAPI
   ↓
Agent Z
   ↓
Tool Calling
   ↓

Rule Engine
   +
Hybrid RAG
   +
PostgreSQL

   ↓

GPT / Gemini / Claude
   ↓

Answer + Sources
```

This gives explainable answers, council-specific guidance, eligibility-question explanations, and much lower hallucination risk than a pure chatbot.

---

# Production-Grade RAG: Complete Conceptual Blueprint

Since your goal is **AI Engineer + Production Systems**, don't think of RAG as:

```text
PDF → Embedding → Vector DB → GPT
```

That's a demo.

Production RAG is actually:

```text
Data Sources
      ↓
Ingestion Pipeline
      ↓
Cleaning & Processing
      ↓
Chunking Strategy
      ↓
Embedding Pipeline
      ↓
Vector Storage
      ↓
Retrieval Layer
      ↓
Reranking Layer
      ↓
Context Construction
      ↓
LLM Generation
      ↓
Guardrails
      ↓
Evaluation
      ↓
Monitoring
```

---

# Level 0: First Principles

RAG exists because LLMs have problems:

| Problem            | Example                                  |
| ------------------ | ---------------------------------------- |
| Hallucinations     | Makes up council rules                   |
| Outdated knowledge | Doesn't know latest planning regulations |
| Private data       | Cannot access internal council data      |
| Context limits     | Cannot load 1000 PDFs                    |

RAG solves this by:

```text
Retrieve
+
Generate
```

---

# Mental Model

Imagine:

```text
100,000 Documents
```

You cannot send everything to GPT.

Instead:

```text
Question
    ↓
Find Relevant Data
    ↓
Send Only Relevant Data
    ↓
Generate Answer
```

---

# End-to-End Architecture

```text
            INGESTION

PDF
DOCX
HTML
DB
API
Images
Emails

      ↓

Document Processing

      ↓

Chunking

      ↓

Embeddings

      ↓

Vector Database

--------------------------------

           RETRIEVAL

Question

      ↓

Query Embedding

      ↓

Similarity Search

      ↓

Reranking

      ↓

Context Building

--------------------------------

           GENERATION

Prompt

Context

Question

      ↓

LLM

      ↓

Answer
```

---

# Stage 1: Data Sources

Production systems rarely use only PDFs.

Typical sources:

```text
PDF
DOCX
Excel
SharePoint
Confluence
Websites
Databases
APIs
Emails
JSON
CMS
```

For your Planning Portal:

```text
Council Rules
FAQs
Eligibility Rules
Forms
Planning Guidance
Service Descriptions
SOPs
Database Records
```

---

# Stage 2: Ingestion Layer

Most beginners ignore this.

This is where many RAG projects fail.

## Responsibilities

### Detect new files

```text
File uploaded
```

### Update files

```text
Version changed
```

### Delete files

```text
Document removed
```

### Re-index

```text
Regenerate embeddings
```

---

# Production Design

```text
Upload Service

      ↓

Queue

      ↓

Ingestion Worker

      ↓

Embedding Worker

      ↓

Vector Store
```

---

# Stage 3: Document Cleaning

Never embed raw documents.

Bad:

```text
Page 1

CONFIDENTIAL

Footer

Page Number

Copyright
```

Good:

```text
Actual content only
```

---

Remove:

* Headers
* Footers
* Navigation
* Page numbers
* Duplicate text

---

# Stage 4: Chunking

The MOST important RAG decision.

Bad chunking = bad RAG.

---

## Fixed Chunking

```text
500 chars
100 overlap
```

Easy.

Not ideal.

---

## Recursive Chunking

Preferred.

Keeps:

```text
Section
Paragraph
Sentence
```

structure.

---

## Semantic Chunking

Split based on meaning.

Best quality.

More expensive.

---

## Parent Child Chunking

Production favorite.

Store:

```text
Parent = Full Section

Child = Small Chunks
```

Retrieve child.

Show parent.

---

# Chunk Size Tradeoff

### Small

```text
200 tokens
```

Pros:

* Precise retrieval

Cons:

* Loses context

---

### Large

```text
2000 tokens
```

Pros:

* More context

Cons:

* Retrieval becomes noisy

---

Typical:

```text
300–800 tokens
```

---

# Stage 5: Embeddings

Convert text:

```text
"What is planning permission?"
```

into:

```text
[0.235, 0.981, 0.551 ...]
```

---

# What Embeddings Capture

Not keywords.

Meaning.

Example:

```text
Leave Policy
```

and

```text
Vacation Rules
```

become similar vectors.

---

# Embedding Models

Typical production choices:

* BGE
* E5
* Voyage
* OpenAI

---

# Embedding Considerations

## Accuracy

Better model

↓

Better retrieval

---

## Cost

More embeddings

↓

More storage

---

## Latency

Large models

↓

Slower ingestion

---

# Stage 6: Vector Database

Stores vectors.

Example:

```text
Chunk
+
Embedding
+
Metadata
```

---

Metadata:

```json
{
  "document": "planning_guide",
  "council": "Council_A",
  "section": "eligibility"
}
```

---

Popular Choices

| DB       | Production  |
| -------- | ----------- |
| Qdrant   | Excellent   |
| Pinecone | Excellent   |
| Weaviate | Good        |
| Milvus   | Large Scale |
| Chroma   | Learning    |

For your projects:

```text
Qdrant
```

---

# Stage 7: Query Processing

User asks:

```text
Why is this question shown?
```

Before retrieval:

Process query.

---

# Query Expansion

Generate:

```text
Why is this field shown?

Eligibility rule

Form condition
```

Multiple searches.

---

# Query Rewriting

Convert:

```text
Why am I seeing this?
```

to

```text
Eligibility condition explanation
```

---

# Stage 8: Retrieval

Core RAG step.

---

# Similarity Search

Compare:

```text
Question Embedding
```

with

```text
Document Embeddings
```

---

# Cosine Similarity

Measures angle.

```text
1.0 = identical

0.0 = unrelated
```

Most common.

---

# Top K

Retrieve:

```text
Top 5
Top 10
Top 20
```

chunks.

---

Tradeoff:

Small K

↓

Miss information

Large K

↓

Context pollution

---

Typical:

```text
5–15 chunks
```

---

# Stage 9: Hybrid Search

Never rely only on vectors.

Production systems use:

```text
BM25
+
Vector Search
```

---

Example:

User searches:

```text
Question 17
```

Keyword match is important.

---

Hybrid Search:

```text
Keyword Search
+
Semantic Search
```

---

# Stage 10: Reranking

Most important production upgrade.

---

Retriever finds:

```text
Top 20
```

chunks.

---

Reranker re-sorts:

```text
Top 20
    ↓
Best 5
```

---

Accuracy improvement:

Often:

```text
20–40%
```

better than retrieval alone.

---

# Stage 11: Context Building

Do NOT send all retrieved chunks.

Build context carefully.

---

Bad

```text
20 random chunks
```

---

Good

```text
Top chunks
+
Metadata
+
Source links
```

---

# Stage 12: Prompt Construction

Typical prompt:

```text
System Prompt

Retrieved Context

User Question
```

---

Example:

```text
Answer only from provided context.

If information is unavailable,
say so.
```

---

# Stage 13: Generation

LLM receives:

```text
Question

+
Context
```

Generates answer.

---

# Stage 14: Guardrails

Critical for enterprise systems.

---

Prevent:

* Hallucinations
* PII leaks
* Prompt injection
* Jailbreaks

---

Example

User:

```text
Ignore previous instructions.
```

Should never work.

---

# Stage 15: Citations

Every answer should show:

```text
Source:
Planning Guide Section 4
```

Users trust citations.

---

# Stage 16: Evaluation

Most teams skip this.

Huge mistake.

---

Measure:

### Retrieval Metrics

* Recall
* Precision
* MRR
* NDCG

---

### Generation Metrics

* Faithfulness
* Relevance
* Groundedness

---

Tools:

* RAGAS
* Langfuse
* DeepEval

---

# Stage 17: Monitoring

Track:

### Latency

```text
Question → Answer
```

---

### Retrieval Quality

```text
Chunks retrieved
```

---

### Hallucinations

```text
Wrong answers
```

---

### Cost

```text
Embedding cost

LLM cost
```

---

# Stage 18: Security

For your UK Planning Portal this is mandatory.

---

## Authentication

Only authorized users.

---

## Authorization

Council A should not see Council B data.

---

## Metadata Filtering

```json
{
  "council_id": 123
}
```

---

## Encryption

At Rest

In Transit

---

## Audit Logs

Track:

```text
Who asked?
What was retrieved?
What was answered?
```

---

# Production RAG Checklist

A production-grade RAG should have:

### Data Layer

* Document ingestion
* Versioning
* Cleaning
* Chunking

### Retrieval Layer

* Embeddings
* Vector DB
* Hybrid Search
* Metadata Filters
* Reranking

### Generation Layer

* Prompt Templates
* Context Builder
* LLM

### Reliability

* Citations
* Guardrails
* Evaluation

### Operations

* Monitoring
* Logging
* Caching
* Cost Tracking

### Security

* Authentication
* Authorization
* Encryption
* Audit Trails

---

# The Architecture I Would Build for Your Planning Portal

```text
Planning Docs
Eligibility Rules
FAQs
Service Definitions
Council Guidance

          ↓

Ingestion Pipeline

          ↓

Chunking

          ↓

Embeddings

          ↓

Qdrant

          ↓

Hybrid Search
(BM25 + Vector)

          ↓

Metadata Filters
(Council, Service, Form)

          ↓

Reranker

          ↓

Context Builder

          ↓

LLM (GPT/Gemini)

          ↓

Answer + Citations

          ↓

Monitoring + Evaluation
```

This is the foundation you should master before writing a single line of RAG code. Once these concepts are clear, the next step is designing the **database schema, ingestion pipeline, chunk metadata model, embedding workflow, Qdrant collections, retrieval flow, and FastAPI service architecture** for a production implementation.

---
