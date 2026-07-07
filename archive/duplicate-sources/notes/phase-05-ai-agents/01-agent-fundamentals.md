# Agent Fundamentals

## 1. Problem Statement

AI agents solve the problem of using LLMs for tasks that require multiple steps, decisions, tools, and state.

A normal LLM call is request-response: user asks, model answers. That is enough for many tasks. But some workflows need the system to plan, call tools, inspect results, update state, and continue until the goal is done.

Without agent architecture:

- complex tasks become one huge fragile prompt
- tool use is hard to coordinate
- intermediate state is lost
- workflows cannot be resumed or audited
- systems may act without clear boundaries

Real-world analogy: a chatbot is like someone answering one question. An agent is like an assistant who can plan a task, check files, call services, remember progress, and report back.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | An AI agent is a system where an LLM helps decide actions toward a goal using tools, state, and control logic. |
| Key terminology | goal, plan, action, tool, observation, state, memory, reflection, loop, stop condition |
| Simple explanation | An agent does not only answer; it can decide what step to take next. |
| Mental model | Goal -> think/plan -> act -> observe -> continue or stop. |
| Easy example | A research agent searches docs, reads sources, and writes a report. |
| Use When | The task needs planning, tools, or multi-step progress. |
| Avoid When | A simple prompt, RAG query, or deterministic workflow is enough. |
| Advantages | Handles flexible, multi-step tasks. |
| Tradeoffs | More cost, latency, testing difficulty, and safety risk. |
| Limitations | Agents can loop, choose wrong tools, or produce unsupported conclusions. |
| Production Example | A workflow agent checks CRM data, drafts a customer follow-up, and waits for approval before sending. |
| Interview Answer | Agents combine LLM decision-making with tools, state, and orchestration to complete multi-step goals under controlled boundaries. |

## 3. Intermediate Explanation

Core agent loop:

```text
Goal -> plan -> action/tool call -> observation -> update state -> next action -> final answer
```

Main components:

| Component | Role |
| --- | --- |
| Goal | What the agent should accomplish |
| Planner | Decides steps or next action |
| Tool executor | Runs approved tools |
| Observation | Result of a tool or step |
| State | Current workflow data |
| Memory | Stored information from previous steps or sessions |
| Critic/reflection | Reviews progress or output |
| Stop condition | Defines when the agent must stop |
| Trace | Auditable record of steps |

Agent types:

| Type | Description | Example |
| --- | --- | --- |
| Reactive agent | Chooses next action based on current state | tool-using chatbot |
| Planner-executor | Creates plan, then executes steps | research assistant |
| Workflow agent | Follows controlled graph with LLM decisions in some nodes | CRM automation |
| Multi-agent system | Multiple role-based agents coordinate | code review team simulation |

## 4. Advanced Explanation

Production agents are orchestration systems, not magical autonomous brains.

Optimization techniques:

- define clear goals and success criteria
- keep tools narrow and typed
- use max step limits
- separate planning from execution
- persist state after each step
- add approval for side effects
- use retrieval for grounding
- log traces for every run
- evaluate task success, not only answer quality

Performance considerations:

- every step may call an LLM or tool
- agents can multiply latency and cost
- tool failures must be handled
- long traces can exceed context windows

Scaling considerations:

- run long agents asynchronously
- store runs and steps in PostgreSQL
- keep active state in Redis if needed
- add worker queues
- set per-tenant cost budgets
- monitor loop/failure rates

Production challenges:

- infinite loops
- unclear stop criteria
- bad tool selection
- hallucinated tool results
- unauthorized actions
- hard evaluation
- user overtrust

## 5. Internal Working

```text
User goal
  |
  v
Agent initializes state
  |
  v
Planner selects next step
  |
  v
Tool call or model step runs
  |
  v
Observation is stored
  |
  v
Stop condition checked
  |
  +--> continue loop
  |
  v
Final answer or failure response
```

Detailed lifecycle:

1. User submits a goal.
2. System authenticates user and creates a run ID.
3. Agent state is initialized.
4. Planner chooses first action.
5. Backend validates action and permissions.
6. Tool or model step executes.
7. Observation is saved.
8. Agent decides whether to continue.
9. Stop condition ends the run.
10. Final answer, trace, and metrics are stored.

## 6. When To Use

Use agents when:

- the task requires multiple steps
- the next step depends on previous results
- tools must be called dynamically
- the workflow needs state
- research or investigation is required
- the user goal is broad and cannot be solved in one call

Ideal use cases:

- research assistant
- coding assistant
- incident investigation assistant
- CRM workflow assistant
- document analysis workflow
- support troubleshooting assistant

## 7. When NOT To Use

Avoid agents when:

- a single prompt works
- RAG can answer directly
- deterministic code is safer
- latency must be low
- tool side effects are high-risk
- you cannot monitor and control the workflow

Better alternatives:

- simple LLM call
- RAG query
- rules engine
- deterministic workflow
- background job
- human-in-the-loop process

## 8. Advantages

- Can adapt to intermediate results.
- Supports tool-using workflows.
- Handles broad goals.
- Can decompose complex tasks.
- Makes multi-step AI applications possible.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Flexibility vs control | Agents adapt, but deterministic systems are easier to guarantee. |
| Capability vs cost | More steps mean more LLM and tool calls. |
| Autonomy vs safety | More autonomy requires stronger permissions and approval. |
| Debuggability vs complexity | Traces help, but agent behavior is harder to reason about. |

## 10. Limitations

- Agents can loop.
- Agents can choose wrong tools.
- Agents can misuse stale or bad memory.
- Agents are harder to test than normal APIs.
- Agent success can be subjective.
- Production agents need strict boundaries.

## 11. Real-World Examples

Startup example: an agent researches competitors and drafts a market summary.

Enterprise example: an agent triages support tickets, checks docs, drafts replies, and asks for approval.

FAANG-style example: a coding assistant inspects repository context, proposes changes, runs checks, and summarizes results.

Production system: a workflow agent uses max steps, typed tools, PostgreSQL traces, Redis rate limits, and human approval for write actions.

## 12. Architecture Diagram

```text
[User Goal]
    |
    v
[Agent Orchestrator] <-> [State Store]
    |
    +--> [Planner LLM]
    +--> [Retriever]
    +--> [Tool Executor]
    +--> [Critic / Validator]
    |
    v
[Final Answer + Trace]
```

## 13. Python Implementation

Agent state:

```python
from dataclasses import dataclass, field

@dataclass
class AgentState:
    run_id: str
    goal: str
    steps: list[str] = field(default_factory=list)
    observations: list[str] = field(default_factory=list)
    final_answer: str | None = None
    failed_reason: str | None = None

    @property
    def done(self) -> bool:
        return self.final_answer is not None or self.failed_reason is not None
```

Stop condition:

```python
MAX_STEPS = 6

def can_continue(state: AgentState) -> bool:
    return not state.done and len(state.steps) < MAX_STEPS
```

Simple agent loop:

```python
def run_simple_agent(state: AgentState) -> AgentState:
    while can_continue(state):
        step = f"step-{len(state.steps) + 1}: inspect available context"
        state.steps.append(step)
        state.observations.append("demo observation")

        if len(state.steps) >= 2:
            state.final_answer = "Completed demo agent workflow."

    if not state.done:
        state.failed_reason = "max steps reached"

    return state
```

## 14. FastAPI Implementation

```python
from uuid import uuid4
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class AgentRunRequest(BaseModel):
    goal: str = Field(min_length=1)

class AgentRunResponse(BaseModel):
    run_id: str
    status: str
    steps: list[str]
    final_answer: str | None

@app.post("/agents/run", response_model=AgentRunResponse)
async def run_agent(request: AgentRunRequest) -> AgentRunResponse:
    state = AgentState(run_id=str(uuid4()), goal=request.goal)
    result = run_simple_agent(state)
    return AgentRunResponse(
        run_id=result.run_id,
        status="completed" if result.final_answer else "failed",
        steps=result.steps,
        final_answer=result.final_answer,
    )
```

Production-ready structure:

```text
app/
  api/routes/agents.py
  services/agent_orchestrator.py
  services/planner.py
  services/tool_executor.py
  services/agent_evaluator.py
  repositories/agent_run_repository.py
```

## 15. Database Integration

PostgreSQL:

```text
agent_runs(id, user_id, tenant_id, goal, status, started_at, finished_at)
agent_steps(id, run_id, step_number, action, observation, latency_ms, cost)
agent_outputs(id, run_id, final_answer, failed_reason, created_at)
```

Redis:

- active run state
- step counters
- rate limits
- distributed locks

Vector DB:

- retrieve memory or documents during agent runs

## 16. Production Considerations

- Set max steps.
- Set max cost.
- Set timeout per step.
- Validate all tool calls.
- Log every step.
- Persist run state.
- Support cancellation.
- Require approval for side effects.
- Use idempotency for write tools.
- Monitor success rate, loop rate, latency, and cost.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Calling every chatbot an agent | Agents need goals, actions, state, or tools |
| Beginner | Using agents for simple tasks | Start with prompt or RAG first |
| Intermediate | No max steps | Add stop conditions |
| Intermediate | No state persistence | Store run and step data |
| Production | Letting agents execute risky actions directly | Add authorization and approval |
| Production | No trace logs | Persist steps, observations, and tool calls |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is an AI agent? | A system where an LLM helps decide actions toward a goal using tools, state, and control logic. |
| Basic | How is an agent different from a chatbot? | A chatbot may answer directly; an agent can plan, use tools, update state, and continue across steps. |
| Intermediate | What is an agent loop? | Repeated plan/action/observation steps until success, failure, or a stop condition. |
| Advanced | How do you make agents production-ready? | Bound steps and cost, validate tools, persist state, log traces, enforce permissions, add approvals, and evaluate task success. |
| Scenario | Agent loops forever. | Add max steps, better stop criteria, state checks, timeout budgets, and failure fallback. |

## 19. System Design Discussion

Agents fit into large systems as workflow orchestrators. The LLM may decide steps, but backend systems must enforce security, tool permissions, state, and observability.

Design decisions:

- one-shot agent vs workflow graph
- tool set
- state schema
- memory policy
- stop criteria
- approval requirements
- async or sync execution
- trace storage

## 20. Hands-On Assignment

- Easy: Define an `AgentState` model.
- Medium: Implement a max-step loop.
- Hard: Store agent run steps and final status in a repository interface.

## 21. Mini Project

Build a Bounded Research Agent Demo.

Requirements:

- Accept a research goal.
- Create a run ID.
- Execute at most 5 steps.
- Store steps and observations.
- Return final answer or max-step failure.

Folder structure:

```text
bounded-agent/
  app/
    main.py
    agent_state.py
    orchestrator.py
    schemas.py
  tests/
    test_agent_limits.py
```

## 22. Production-Level Project

Build a Workflow Agent Platform.

Real-world problem:

- A company needs agents that perform internal workflows safely and audibly.

Architecture:

```text
[User] -> [FastAPI] -> [Agent Orchestrator]
                         |
                         +-> [Planner]
                         +-> [Tool Executor]
                         +-> [State Store]
                         +-> [Approval Service]
                         +-> [Trace Logger]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- vector DB for knowledge retrieval
- background workers
- LLM API

Scaling strategy:

- queue long-running agents
- rate-limit by tenant
- persist checkpoints
- monitor cost per run
- require approval for side effects
- evaluate task success with test scenarios

## Quiz

1. What is an AI agent?
2. How is an agent different from a single LLM call?
3. What is an agent loop?
4. What is agent state?
5. What is an observation?
6. Why do agents need stop conditions?
7. Why are tool permissions important?
8. What should be stored in an agent trace?
9. When should you avoid agents?
10. How would you design a production-safe agent?

## Knowledge Check

You should be able to explain agent fundamentals, identify when agents are useful, and design a bounded agent loop with state, tools, traces, and stop conditions.

Are you ready for the next section?
