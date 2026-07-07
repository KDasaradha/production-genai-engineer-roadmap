# Graph RAG and Agentic RAG

## 1. Problem Statement

Graph RAG and Agentic RAG solve retrieval problems that require relationships, multi-hop reasoning, planning, or dynamic retrieval steps.

Basic RAG usually retrieves chunks similar to a query. That works for many Q&A systems, but some questions need relationship traversal or multiple search actions.

Examples:

- "Which projects depend on the service affected by this incident?"
- "Compare this contract clause with our standard policy and find related exceptions."
- "Research this topic, inspect several sources, and produce a report."

Without advanced RAG patterns:

- relationship-heavy answers are missed
- multi-document reasoning is weak
- retrieval does not adapt to intermediate findings
- agents may retrieve blindly or loop without control

Real-world analogy: basic RAG is searching pages in a book. Graph RAG is following relationship links in a map. Agentic RAG is a researcher planning which sources to inspect next.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Graph RAG uses entity relationships for retrieval; Agentic RAG uses an agent workflow to plan and perform retrieval steps. |
| Key terminology | entity, relationship, graph, node, edge, multi-hop retrieval, planner, tool, state |
| Simple explanation | Graph RAG follows connections; Agentic RAG decides retrieval steps dynamically. |
| Mental model | Use a map of relationships or a controlled researcher workflow. |
| Easy example | Find documents connected to a product, team, incident, and service. |
| Use When | Basic chunk retrieval misses relationships or multi-step context. |
| Avoid When | Basic RAG solves the problem reliably. |
| Advantages | Better for relationship-aware and multi-step questions. |
| Tradeoffs | More complexity, latency, evaluation difficulty, and data modeling. |
| Limitations | Graph quality and agent control are hard. |
| Production Example | Enterprise knowledge assistant connects teams, services, incidents, docs, and owners. |
| Interview Answer | Graph RAG extends RAG with entity relationships, while Agentic RAG lets a controlled agent plan retrieval steps using tools and state. |

## 3. Intermediate Explanation

Graph RAG components:

- entity extraction
- relationship extraction
- graph store
- node and edge metadata
- graph traversal
- vector search integration
- answer synthesis

Agentic RAG components:

- planner
- retriever tools
- state store
- step limit
- observations
- answer writer
- trace logs

Comparison:

| Pattern | Best For | Main Risk |
| --- | --- | --- |
| Basic RAG | direct document Q&A | misses relationships |
| Graph RAG | entity and relationship questions | graph extraction quality |
| Agentic RAG | dynamic research workflows | loops, cost, tool misuse |
| Graph + Agentic RAG | complex enterprise research | high complexity |

Data flow:

```text
Question -> entity detection -> graph/vector retrieval -> synthesis
Question -> agent plan -> retrieval tools -> observations -> final answer
```

## 4. Advanced Explanation

Graph RAG should be justified by relationship-heavy questions. Agentic RAG should be justified by workflows where one retrieval step informs the next.

Optimization techniques:

- start with basic RAG baseline
- add graph only for known relationship failures
- extract entities with confidence scores
- keep graph schema simple
- combine graph traversal with vector retrieval
- bound agent steps
- log every agent action
- require citations from retrieved sources

Performance considerations:

- graph extraction adds ingestion cost
- graph traversal can be fast if schema is clean
- agent loops add unpredictable latency
- multi-hop retrieval increases token usage

Scaling considerations:

- version graph extraction prompts/models
- update graph when documents change
- store run traces for agent workflows
- set budgets for agent retrieval
- evaluate multi-hop questions separately

Production challenges:

- incorrect entity extraction
- duplicate entities
- stale relationships
- agent loops
- weak stop criteria
- graph and vector results disagree
- hard evaluation

## 5. Internal Working

Graph RAG lifecycle:

```text
Documents
  |
  v
Extract entities and relationships
  |
  v
Store graph nodes and edges
  |
  v
Question maps to entities
  |
  v
Traverse graph and retrieve related chunks
  |
  v
LLM synthesizes answer with citations
```

Agentic RAG lifecycle:

```text
Goal
  |
  v
Planner decides next retrieval step
  |
  v
Retriever/search tool runs
  |
  v
Observation saved in state
  |
  v
Continue or stop
  |
  v
Final grounded answer
```

## 6. When To Use

Use Graph RAG when:

- relationships matter
- questions involve entities and dependencies
- multi-hop answers are common
- documents reference each other

Use Agentic RAG when:

- retrieval path is not known upfront
- the system must search, inspect, and refine
- user asks broad research questions
- multiple tools or corpora are involved

## 7. When NOT To Use

Avoid Graph or Agentic RAG when:

- basic RAG answers well
- corpus is small
- latency must be low
- relationships are not reliable
- team cannot evaluate complex workflows

Better alternatives:

- improve chunking
- use hybrid retrieval
- use reranking
- use metadata filters
- use a deterministic workflow instead of an agent

## 8. Advantages

- Handles relationship-aware questions.
- Supports multi-hop retrieval.
- Improves research workflows.
- Can combine structured and unstructured knowledge.
- Gives richer enterprise context.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Capability vs complexity | Advanced RAG handles harder questions but is harder to build. |
| Autonomy vs control | Agentic RAG adapts but needs limits. |
| Graph quality vs answer quality | Bad graph data causes wrong paths. |
| Multi-hop depth vs latency | More hops increase cost and time. |

## 10. Limitations

- Graph extraction can be wrong.
- Entity resolution is difficult.
- Agentic workflows can loop.
- Evaluation is harder than basic RAG.
- More components mean more failure points.
- Not necessary for many practical RAG apps.

## 11. Real-World Examples

Startup example: a research assistant searches multiple docs and writes a cited report.

Enterprise example: an incident assistant connects services, owners, runbooks, previous incidents, and deployment records.

FAANG-style example: a knowledge system combines graph relationships, vector retrieval, ranking, and workflow planning.

Production system: a compliance assistant uses graph traversal to find related policies, then vector retrieval to fetch exact evidence.

## 12. Architecture Diagram

Graph RAG:

```text
[Documents] -> [Entity Extractor] -> [Graph Store]
      |                                  |
      v                                  v
[Vector DB] <-------------------- [Graph Traversal]
      |
      v
[Context Builder] -> [LLM]
```

Agentic RAG:

```text
[User Goal] -> [Planner] -> [Retrieval Tool]
                  ^              |
                  |              v
              [State Store] <- [Observation]
                  |
                  v
             [Final Answer]
```

## 13. Python Implementation

Graph edge:

```python
from dataclasses import dataclass

@dataclass
class Entity:
    id: str
    name: str
    type: str

@dataclass
class Relationship:
    source_id: str
    relation: str
    target_id: str
```

Simple traversal:

```python
def neighbors(entity_id: str, relationships: list[Relationship]) -> list[str]:
    return [
        relationship.target_id
        for relationship in relationships
        if relationship.source_id == entity_id
    ]
```

Agentic RAG state:

```python
from dataclasses import field

@dataclass
class AgenticRagState:
    question: str
    steps: list[str] = field(default_factory=list)
    observations: list[str] = field(default_factory=list)
    final_answer: str | None = None

def can_continue(state: AgenticRagState, max_steps: int) -> bool:
    return state.final_answer is None and len(state.steps) < max_steps
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class AgenticRagRequest(BaseModel):
    question: str = Field(min_length=1)
    max_steps: int = Field(default=4, ge=1, le=10)

class AgenticRagResponse(BaseModel):
    answer: str
    steps: list[str]

@app.post("/rag/agentic", response_model=AgenticRagResponse)
async def agentic_rag(request: AgenticRagRequest) -> AgenticRagResponse:
    state = AgenticRagState(question=request.question)
    state.steps.append("analyze question")
    state.observations.append("basic retrieval is enough for demo")
    state.final_answer = "This endpoint would run bounded retrieval steps in production."
    return AgenticRagResponse(answer=state.final_answer, steps=state.steps)
```

Production-ready structure:

```text
app/
  services/graph_extractor.py
  services/graph_retriever.py
  services/agentic_rag_orchestrator.py
  services/retrieval_tools.py
  repositories/graph_repository.py
  repositories/agent_trace_repository.py
```

## 15. Database Integration

Graph tables:

```text
entities(id, tenant_id, name, type, confidence)
relationships(id, tenant_id, source_id, relation, target_id, confidence)
entity_sources(id, entity_id, document_id, chunk_id)
```

Agent trace tables:

```text
agentic_rag_runs(id, user_id, question, status, started_at, finished_at)
agentic_rag_steps(id, run_id, step_number, action, observation, cost, latency_ms)
```

Vector DB:

- stores chunks linked to entities
- supports semantic evidence retrieval

Redis:

- active agent run state
- step limits and rate limits

## 16. Production Considerations

- Start with basic RAG baseline.
- Add Graph RAG only for relationship failures.
- Add Agentic RAG only when dynamic retrieval is needed.
- Store graph extraction confidence.
- Deduplicate entities.
- Log all agent steps.
- Limit agent steps and cost.
- Require citations for final answers.
- Evaluate multi-hop questions separately.
- Avoid uncontrolled write tools.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Starting with Graph RAG before basic RAG | Build and evaluate basic RAG first |
| Beginner | Calling any RAG app "agentic" | Use agentic only when retrieval steps are planned dynamically |
| Intermediate | No entity deduplication | Add entity resolution rules |
| Intermediate | No max steps | Bound agent loops |
| Production | No graph/version updates | Rebuild graph when documents change |
| Production | No trace logs | Persist every agent action and observation |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is Graph RAG? | RAG that uses entities and relationships to retrieve connected context. |
| Basic | What is Agentic RAG? | RAG where an agent plans and performs retrieval steps dynamically. |
| Intermediate | When use Graph RAG? | When answers depend on relationships, dependencies, or multi-hop context. |
| Advanced | What are the risks? | Graph quality problems, entity duplication, agent loops, latency, cost, and evaluation complexity. |
| Scenario | Basic RAG misses dependency questions. | Add entity extraction, graph relationships, graph traversal, and linked evidence chunks. |

## 19. System Design Discussion

Graph RAG and Agentic RAG are advanced extensions, not default starting points.

Design decisions:

- graph schema
- entity extraction method
- entity resolution
- graph store choice
- vector plus graph retrieval strategy
- agent max steps
- tool boundaries
- trace storage
- evaluation set for multi-hop questions

## 20. Hands-On Assignment

- Easy: Model entities and relationships for a small project system.
- Medium: Build a function that finds related entities.
- Hard: Design a bounded Agentic RAG flow with max steps and trace logs.

## 21. Mini Project

Build a Project Dependency Graph Assistant.

Requirements:

- Store services, teams, owners, and dependencies.
- Retrieve related entities.
- Link entities to document chunks.
- Answer dependency questions with citations.

Folder structure:

```text
graph-rag-demo/
  app/
    main.py
    graph.py
    graph_retriever.py
    schemas.py
  tests/
    test_graph_retriever.py
```

## 22. Production-Level Project

Build an Enterprise Relationship-Aware RAG Assistant.

Real-world problem:

- Employees need answers that connect documents, teams, services, policies, incidents, and owners.

Architecture:

```text
[Documents] -> [Chunker] -> [Vector DB]
            -> [Entity Extractor] -> [Graph Store]

[Question] -> [Entity Resolver]
            -> [Graph Retriever]
            -> [Vector Retriever]
            -> [Context Builder]
            -> [LLM Answer]
```

Tech stack:

- FastAPI
- PostgreSQL or graph database
- Vector DB
- Redis
- LLM API
- background extraction workers

Scaling strategy:

- update graph asynchronously
- version extraction model and schema
- deduplicate entities
- cache graph traversals
- trace multi-hop retrieval
- evaluate relationship questions separately

## Quiz

1. What is Graph RAG?
2. What is Agentic RAG?
3. What is an entity?
4. What is a relationship or edge?
5. What is multi-hop retrieval?
6. Why start with basic RAG first?
7. Why can graph extraction fail?
8. Why do agentic systems need max steps?
9. What should be stored in an agent trace?
10. How do you combine graph retrieval and vector retrieval?

## Knowledge Check

You should be able to explain when Graph RAG or Agentic RAG is justified, and design safe boundaries for relationship-aware or multi-step retrieval.

Are you ready for the next section?
