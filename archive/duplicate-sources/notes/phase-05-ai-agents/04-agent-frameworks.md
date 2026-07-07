# Agent Frameworks

## 1. Problem Statement

Agent frameworks solve the problem of building stateful, tool-using, multi-step AI workflows without wiring every orchestration detail from scratch.

Frameworks can help with graphs, tools, memory, agent roles, traces, and workflow control. But they do not automatically make agents safe or production-ready.

Without frameworks:

- complex workflow code can become repetitive
- state transitions may be hand-rolled poorly
- tool orchestration may lack structure

With frameworks used poorly:

- hidden behavior is hard to debug
- tool permissions may be unclear
- framework lock-in grows
- tutorials get copied into production without boundaries

Real-world analogy: a framework is like a workflow engine. It helps organize steps, but you still need business rules, safety checks, and monitoring.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Agent frameworks provide abstractions for building LLM workflows with tools, state, memory, and multiple steps. |
| Key terminology | graph, node, edge, tool, agent role, state, memory, trace, orchestration, MCP |
| Simple explanation | Frameworks help structure agent workflows. |
| Mental model | Use the framework to organize the workflow, not to avoid understanding it. |
| Easy example | LangGraph represents an agent as stateful nodes and transitions. |
| Use When | Workflow state, tools, or branching are complex enough to justify abstraction. |
| Avoid When | A simple service function is clearer. |
| Advantages | Faster development and reusable orchestration patterns. |
| Tradeoffs | Lock-in, learning curve, and debugging complexity. |
| Limitations | Frameworks do not solve safety, evaluation, or permissions automatically. |
| Production Example | An agent service wraps LangGraph behind internal interfaces and stores traces in PostgreSQL. |
| Interview Answer | Agent frameworks help structure tool-using workflows, but production reliability comes from clear state, tool boundaries, observability, tests, and permissions. |

## 3. Intermediate Explanation

Common frameworks and ecosystems:

| Framework | Strength | Typical Use |
| --- | --- | --- |
| LangGraph | stateful graph workflows | controlled agents and agentic RAG |
| LangChain | tools, chains, integrations | prototypes and tool workflows |
| CrewAI | role-based multi-agent workflows | collaborative agent demos/workflows |
| AutoGen | multi-agent conversation patterns | research and agent collaboration |
| Semantic Kernel | enterprise orchestration | Microsoft ecosystem |
| MCP | protocol for tools/context | standardized tool access |

Framework concepts:

- node: one step in a workflow
- edge: transition between steps
- state: data passed between nodes
- tool: external capability
- memory: persisted context
- trace: log of execution
- checkpoint: saved workflow state

Data flow:

```text
Input -> framework graph/workflow -> model/tool nodes -> state updates -> final output
```

## 4. Advanced Explanation

Production framework usage should be replaceable, observable, and bounded.

Optimization techniques:

- wrap framework code behind service interfaces
- keep route handlers framework-free
- define your own state schemas
- persist traces outside the framework
- pin framework versions
- write contract tests
- centralize tool permission checks
- avoid uncontrolled multi-agent loops
- keep human approval outside the model

Performance considerations:

- framework abstractions add overhead
- multi-agent workflows multiply model calls
- graph checkpoints add storage writes
- tracing can add cost but saves debugging time

Scaling considerations:

- run long workflows in workers
- store checkpoints durably
- monitor per-node latency and failures
- rate-limit expensive workflows
- keep framework adapters isolated

Production challenges:

- fast-changing APIs
- hidden prompts
- unclear retry behavior
- hard-to-debug multi-agent conversations
- framework-specific state formats
- migrating away later

## 5. Internal Working

```text
Application request
  |
  v
Your agent service
  |
  v
Framework workflow:
  - state initialization
  - node execution
  - tool calls
  - transitions
  - checkpoints
  |
  v
Your validation and persistence
  |
  v
API response
```

Detailed lifecycle:

1. API receives user goal.
2. Your service creates state.
3. Framework graph starts.
4. Node calls LLM or tool.
5. State is updated.
6. Transition chooses next node.
7. Checkpoint is saved.
8. Workflow ends or fails.
9. Your code validates and stores final result.
10. Trace is available for review.

## 6. When To Use

Use agent frameworks when:

- workflow has branching
- state transitions are complex
- tools are numerous
- checkpointing is needed
- agentic RAG is being built
- team benefits from graph abstractions
- rapid experimentation is valuable

Ideal use cases:

- research agents
- code assistants
- multi-step RAG
- workflow automation
- tool orchestration
- stateful assistants

## 7. When NOT To Use

Avoid frameworks when:

- workflow is simple
- one or two service functions are enough
- team cannot debug the framework
- production constraints are strict and hidden behavior is risky
- framework adds more complexity than value

Better alternatives:

- custom orchestrator
- deterministic workflow engine
- background job queue
- simple FastAPI service
- direct tool execution

## 8. Advantages

- Faster workflow construction.
- Built-in graph or agent patterns.
- Tool integration helpers.
- Checkpointing support in some frameworks.
- Easier experimentation.
- Good learning path for advanced agents.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Speed vs control | Frameworks speed development but may hide behavior. |
| Abstraction vs debugging | Graphs help structure but can be hard to inspect. |
| Multi-agent power vs cost | Multiple agents can multiply token usage. |
| Integration vs lock-in | Framework APIs can shape your architecture too much. |

## 10. Limitations

- Frameworks do not guarantee correct agent behavior.
- They do not replace backend authorization.
- They do not solve prompt injection automatically.
- They can change quickly.
- They can make simple workflows harder.
- They still need testing and monitoring.

## 11. Real-World Examples

Startup example: a team prototypes a research agent using LangGraph.

Enterprise example: an internal workflow agent uses framework graph nodes but stores run state and approvals in company databases.

FAANG-style example: internal platforms often build framework-like orchestration but centralize tools, tracing, security, and evals.

Production system: a FastAPI agent service wraps framework workflows behind an `AgentOrchestrator` interface and logs every node transition.

## 12. Architecture Diagram

```text
[FastAPI]
    |
    v
[AgentOrchestrator Interface]
    |
    +-> [LangGraph Adapter]
    +-> [Custom Workflow Adapter]
    |
    v
[Tool Gateway] -> [Policy Checks] -> [Tools]
    |
    v
[Trace Store + Checkpoints]
```

Good boundary:

```text
Your app depends on your interfaces.
Only adapters depend on agent frameworks.
```

## 13. Python Implementation

Internal orchestrator interface:

```python
from dataclasses import dataclass

@dataclass
class AgentRunResult:
    run_id: str
    status: str
    final_answer: str | None
    steps: list[str]

class AgentOrchestrator:
    def run(self, goal: str, user_id: str) -> AgentRunResult:
        raise NotImplementedError
```

Framework adapter shape:

```python
class FrameworkAgentAdapter(AgentOrchestrator):
    def __init__(self, workflow: object) -> None:
        self.workflow = workflow

    def run(self, goal: str, user_id: str) -> AgentRunResult:
        # In production, call framework workflow and map result to your schema.
        return AgentRunResult(
            run_id="demo",
            status="completed",
            final_answer="framework result",
            steps=["start", "finish"],
        )
```

Custom fallback:

```python
class SimpleAgentOrchestrator(AgentOrchestrator):
    def run(self, goal: str, user_id: str) -> AgentRunResult:
        return AgentRunResult(
            run_id="simple",
            status="completed",
            final_answer=f"Handled goal: {goal}",
            steps=["single-step"],
        )
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel, Field

app = FastAPI()

class AgentFrameworkRequest(BaseModel):
    goal: str = Field(min_length=1)
    user_id: str

class AgentFrameworkResponse(BaseModel):
    run_id: str
    status: str
    final_answer: str | None
    steps: list[str]
    backend: str

def get_orchestrator() -> AgentOrchestrator:
    return SimpleAgentOrchestrator()

@app.post("/agents/framework-run", response_model=AgentFrameworkResponse)
async def framework_run(
    request: AgentFrameworkRequest,
    orchestrator: AgentOrchestrator = Depends(get_orchestrator),
) -> AgentFrameworkResponse:
    result = orchestrator.run(request.goal, request.user_id)
    return AgentFrameworkResponse(
        run_id=result.run_id,
        status=result.status,
        final_answer=result.final_answer,
        steps=result.steps,
        backend=orchestrator.__class__.__name__,
    )
```

Production-ready structure:

```text
app/
  interfaces/agent_orchestrator.py
  adapters/langgraph_agent.py
  adapters/crewai_agent.py
  services/agent_service.py
  services/tool_gateway.py
  repositories/agent_trace_repository.py
  tests/test_agent_orchestrator_contract.py
```

## 15. Database Integration

PostgreSQL:

```text
agent_framework_runs(id, user_id, framework_name, framework_version, status, created_at)
agent_framework_steps(id, run_id, node_name, input_json, output_json, latency_ms)
agent_framework_configs(id, framework_name, config_json, active, created_at)
```

Redis:

- active graph state
- locks
- step counters
- rate limits

Vector DB:

- retrieval nodes in agentic RAG
- memory retrieval

Store framework metadata:

- framework name
- framework version
- graph/workflow config
- adapter version
- model versions

## 16. Production Considerations

- Pin framework versions.
- Wrap framework internals.
- Store traces outside framework when possible.
- Use your own tool gateway.
- Enforce permissions in backend code.
- Add timeout and retry policy per node.
- Add contract tests.
- Keep migration path open.
- Monitor node-level latency and failure rate.
- Avoid uncontrolled multi-agent chat loops.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Starting with multi-agent systems too early | Start with one bounded workflow |
| Beginner | Assuming framework means production-ready | Add auth, logs, tests, limits, and evals |
| Intermediate | Framework code inside routes | Use service and adapter layers |
| Intermediate | No version pinning | Pin and test dependency upgrades |
| Production | No trace export | Store node transitions and tool calls |
| Production | Framework controls permissions | Centralize permissions in backend services |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | Why use an agent framework? | To structure multi-step LLM workflows with tools, state, and transitions. |
| Basic | What is LangGraph useful for? | Building stateful graph-based agent workflows. |
| Intermediate | What is MCP? | A protocol pattern for connecting models or agents to tools and context in a standardized way. |
| Advanced | How do you use frameworks safely in production? | Wrap them behind interfaces, pin versions, centralize tool permissions, persist traces, add tests, and monitor every node. |
| Scenario | Framework upgrade breaks your agent. | Roll back, run contract tests, inspect adapter changes, and keep the framework isolated behind your interface. |

## 19. System Design Discussion

Agent frameworks should be implementation details. Your architecture should own:

- state schema
- tool gateway
- permissions
- memory policy
- trace storage
- response contracts
- evaluation
- deployment and rollback

Principle:

```text
Use frameworks for orchestration speed.
Use your own boundaries for production control.
```

## 20. Hands-On Assignment

- Easy: Compare LangGraph, CrewAI, AutoGen, and MCP.
- Medium: Define an `AgentOrchestrator` interface.
- Hard: Build two orchestrator implementations that pass the same contract test.

## 21. Mini Project

Build a Replaceable Agent Orchestrator Demo.

Requirements:

- Define an orchestrator interface.
- Implement a simple custom orchestrator.
- Implement a framework adapter stub.
- Expose a FastAPI endpoint.
- Add contract tests.

Folder structure:

```text
replaceable-agent/
  app/
    main.py
    interfaces.py
    adapters.py
    schemas.py
  tests/
    test_orchestrator_contract.py
```

## 22. Production-Level Project

Build a Framework-Wrapped Agent Platform.

Real-world problem:

- Team wants to use agent frameworks while preserving security, observability, and replaceability.

Architecture:

```text
[FastAPI] -> [Agent Service]
              |
              +-> [AgentOrchestrator Interface]
              |       +-> [LangGraph Adapter]
              |       +-> [Custom Adapter]
              |
              +-> [Tool Gateway]
              +-> [State Store]
              +-> [Trace Logger]
              +-> [Approval Service]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- vector DB if using memory/RAG
- selected framework
- pytest

Scaling strategy:

- isolate framework adapters
- queue long workflows
- checkpoint state
- trace node transitions
- rate-limit expensive agents
- pin versions and test upgrades

## Quiz

1. Why use agent frameworks?
2. What is LangGraph useful for?
3. What is CrewAI often used for?
4. What is AutoGen often used for?
5. What is MCP?
6. Why wrap framework code behind interfaces?
7. Why should tool permissions live outside the framework?
8. What should be logged for framework workflows?
9. What is framework lock-in?
10. When is a custom orchestrator better?

## Knowledge Check

You should be able to explain agent framework tradeoffs and design a production-safe framework boundary with interfaces, traces, permissions, tests, and rollback.

Are you ready for the next section?
