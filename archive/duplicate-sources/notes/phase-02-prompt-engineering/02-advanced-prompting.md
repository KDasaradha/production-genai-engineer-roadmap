# Advanced Prompting

## 1. Problem Statement

Advanced prompting solves tasks that are too complex for a single direct instruction. Some tasks need planning, decomposition, tool use, critique, or multi-step reasoning control.

If advanced prompting does not exist, developers often force one vague prompt to do everything. That leads to shallow answers, skipped steps, weak tool use, and poor reliability.

Real-world analogy: for a complex project, a senior engineer does not just "start coding." They clarify the goal, break work into steps, inspect data, implement, test, and explain tradeoffs.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Advanced prompting is a set of prompt patterns for complex reasoning, planning, tool use, critique, and multi-step workflows. |
| Key terminology | decomposition, Chain of Thought, Tree of Thought, ReAct, reflection, critique, planner, tool call |
| Simple explanation | Instead of asking the model for one immediate answer, you guide the process it should follow. |
| Mental model | You are giving the model a workflow, not only a question. |
| Easy example | "Break the problem into steps, identify missing information, then answer." |
| Use When | The task needs reasoning, planning, comparison, tool use, or careful review. |
| Avoid When | A simple prompt or deterministic function works. |
| Advantages | Better task structure and more reliable complex workflows. |
| Tradeoffs | More tokens, latency, complexity, and failure modes. |
| Limitations | Reasoning-looking text can still be wrong. |
| Production Example | A research agent plans searches, reads sources, stores notes, and writes a report. |
| Interview Answer | Advanced prompting structures the model's intermediate behavior to improve complex task performance, but production systems still need validation and limits. |

## 3. Intermediate Explanation

Important patterns:

| Pattern | Meaning | Use Case | Risk |
| --- | --- | --- | --- |
| Decomposition | Break task into smaller steps | planning, coding, analysis | unnecessary verbosity |
| Chain of Thought style | Encourage stepwise reasoning | math, logic, planning | reasoning may be wrong or sensitive |
| Tree of Thought style | Explore multiple candidate paths | strategy, alternatives | expensive |
| ReAct | Reason, act with tools, observe, continue | agents, research | loops and tool misuse |
| Reflection/Critique | Review and improve output | writing, code review | can over-correct |
| Self-consistency | Compare multiple answers | uncertain tasks | extra cost |

Practical examples:

- Contract analysis: identify clause, classify risk, explain reason.
- Research: plan search queries, read sources, synthesize answer.
- Debugging: list hypotheses, inspect evidence, rank likely causes.
- Coding assistant: understand file, plan change, edit, test.

Data flow:

```text
Goal -> task decomposition -> step execution -> critique/validation -> final response
```

## 4. Advanced Explanation

Advanced prompting becomes production engineering when it controls actual workflows.

Optimization techniques:

- Use decomposition only for tasks that need it.
- Keep private reasoning separate from user-visible output when possible.
- Use structured intermediate states.
- Add max steps for iterative workflows.
- Validate tool calls with schemas.
- Add stop conditions before model loops begin.
- Store traces for debugging.

Performance considerations:

- Multi-step prompts can increase latency.
- ReAct loops can create many model/tool calls.
- Reflection doubles cost if every answer is reviewed.
- Tree-style exploration can grow quickly.

Scaling considerations:

- Use queues for long-running workflows.
- Persist state after each step.
- Add retry and timeout policies.
- Use smaller models for planning or classification when possible.
- Track cost per workflow run.

Production challenges:

- Infinite or useless loops.
- Tool calls with bad arguments.
- Hard-to-debug intermediate reasoning.
- Users overtrusting confident plans.
- Prompt injection influencing tool decisions.

## 5. Internal Working

```text
Input goal
  |
  v
Planner creates steps or decides next action
  |
  v
Executor runs step or tool
  |
  v
Observation is added to state
  |
  v
Critic validates progress or final answer
  |
  v
Final response returned when stop condition is met
```

Detailed lifecycle:

1. User submits a goal.
2. System builds a prompt with role, tools, constraints, and stop rules.
3. Model creates a plan or chooses next step.
4. Backend validates any action.
5. Tool or model step runs.
6. Result is stored in state.
7. Loop continues until success, failure, or max step limit.
8. Final answer is formatted for the user.

## 6. When To Use

Use advanced prompting for:

- research workflows
- code analysis
- root-cause analysis
- multi-document comparison
- agent tool use
- planning tasks
- scenario analysis
- complex extraction with explanation

## 7. When NOT To Use

Avoid advanced prompting when:

- the task is simple classification
- a database query answers the question
- simple structured extraction works
- latency must be very low
- output must be deterministic

Better alternatives:

- normal code for rules
- SQL for structured data
- RAG for knowledge lookup
- background workflows for long tasks
- fine-tuning only after repeated prompt limits are proven

## 8. Advantages

- Handles complex tasks more systematically.
- Makes multi-step workflows easier to design.
- Can improve answer completeness.
- Supports tool use and agent behavior.
- Helps with debugging by making steps observable.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Quality vs cost | More steps may improve quality but cost more. |
| Flexibility vs control | Agents adapt, but deterministic workflows are easier to guarantee. |
| Reasoning vs privacy | Intermediate reasoning may contain sensitive data. |
| Autonomy vs safety | More autonomy requires stronger permissions and validation. |

## 10. Limitations

- Step-by-step text does not guarantee correct reasoning.
- Complex prompts are harder to test.
- Multi-step workflows can fail halfway.
- Tool-using prompts need backend safety.
- More context can increase prompt injection risk.

## 11. Real-World Examples

Startup example: an AI research assistant creates an outline, searches the web, summarizes sources, and writes a report.

Enterprise example: a compliance assistant reviews a policy, identifies risk areas, and prepares a checklist.

FAANG-style example: a coding assistant uses planning, repository search, test execution, and final explanation.

Production system: a workflow agent uses bounded ReAct steps with max iterations, validated tools, trace logs, and human approval for write actions.

## 12. Architecture Diagram

```text
[User Goal]
    |
    v
[Planner Prompt] -> [State Store]
    |
    v
[Action Selector] -> [Tool Validator] -> [Tool/API]
    |                                      |
    v                                      v
[Observation] <---------------------- [Tool Result]
    |
    v
[Critic / Stop Check]
    |
    v
[Final Answer]
```

## 13. Python Implementation

Decomposition prompt:

```python
def build_decomposition_prompt(task: str) -> str:
    return f"""Break the task into clear steps before answering.

Task:
{task}

Return:
1. Steps
2. Final answer"""
```

ReAct-style control prompt:

```python
def build_react_prompt(goal: str, tools: list[str]) -> str:
    tool_list = ", ".join(tools)
    return f"""You are solving a task with tools.

Goal: {goal}
Available tools: {tool_list}

Rules:
- Use tools only when needed.
- Do not invent tool results.
- Stop when you have enough evidence.
- Return only the final answer to the user."""
```

Workflow state:

```python
from dataclasses import dataclass, field

@dataclass
class WorkflowState:
    goal: str
    steps: list[str] = field(default_factory=list)
    observations: list[str] = field(default_factory=list)
    final_answer: str | None = None

    @property
    def done(self) -> bool:
        return self.final_answer is not None
```

Bounded loop:

```python
MAX_STEPS = 5

def can_continue(state: WorkflowState) -> bool:
    return not state.done and len(state.steps) < MAX_STEPS
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class AdvancedPromptRequest(BaseModel):
    goal: str = Field(min_length=1)
    pattern: str = "decomposition"

class AdvancedPromptResponse(BaseModel):
    prompt: str
    pattern: str
    max_steps: int

@app.post("/prompts/advanced", response_model=AdvancedPromptResponse)
async def build_advanced_prompt(request: AdvancedPromptRequest) -> AdvancedPromptResponse:
    if request.pattern not in {"decomposition", "react"}:
        raise HTTPException(status_code=400, detail="unsupported advanced prompt pattern")

    if request.pattern == "decomposition":
        prompt = build_decomposition_prompt(request.goal)
    else:
        prompt = build_react_prompt(request.goal, tools=["search_docs", "read_source"])

    return AdvancedPromptResponse(prompt=prompt, pattern=request.pattern, max_steps=MAX_STEPS)
```

Production-ready structure:

```text
app/
  api/routes/workflows.py
  services/workflow_orchestrator.py
  services/tool_executor.py
  models/workflow_models.py
  repositories/workflow_run_repository.py
```

## 15. Database Integration

Store:

- workflow run ID
- user ID or tenant ID
- goal
- pattern used
- step number
- action selected
- tool arguments
- observation
- final output
- status
- cost and latency

Useful schema:

```text
workflow_runs(id, user_id, goal, status, started_at, finished_at)
workflow_steps(id, run_id, step_number, action, observation, latency_ms, cost)
```

Redis use:

- temporary workflow state
- locks for active runs
- rate limits for expensive workflows

Vector DB use:

- retrieve context for research or RAG-based workflows

## 16. Production Considerations

- Set max steps.
- Set max cost per run.
- Set timeout per step.
- Validate every tool call.
- Log every decision and observation.
- Make workflows resumable when long-running.
- Require human approval for side effects.
- Keep user-visible answer separate from internal trace.
- Use evaluation tasks to measure success rate.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Using advanced prompting for simple tasks | Start with simple prompts first |
| Beginner | Assuming reasoning text is always true | Validate claims and results |
| Intermediate | No max step limit | Add bounded loops |
| Intermediate | No tool validation | Use schemas and backend permission checks |
| Production | No workflow trace | Persist steps and observations |
| Production | Letting agents perform risky writes automatically | Add human approval and idempotency |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is advanced prompting? | Prompt patterns that guide complex reasoning, planning, critique, or tool use. |
| Intermediate | What is ReAct? | A pattern where the model reasons, chooses an action, observes tool results, and continues until it can answer. |
| Intermediate | What is task decomposition? | Breaking a large task into smaller steps before solving it. |
| Advanced | What are production risks of advanced prompting? | Higher cost, latency, loops, tool misuse, prompt injection, and hard-to-debug traces. |
| Scenario | An agent keeps looping. What do you do? | Add max steps, clear stop criteria, state checks, tool result validation, and failure fallback. |

## 19. System Design Discussion

Advanced prompting is the bridge between simple LLM calls and agent systems. The main design decision is whether the task should be:

- one model call
- a deterministic workflow
- a RAG pipeline
- an agentic tool loop

For production, prefer the simplest architecture that meets reliability requirements. Use advanced prompting only where the task complexity pays for the added cost and risk.

## 20. Hands-On Assignment

- Easy: Write a decomposition prompt for debugging a slow API.
- Medium: Write a critique prompt that reviews an answer for missing evidence.
- Hard: Design a bounded ReAct loop with max steps, tool schemas, and stop conditions.

## 21. Mini Project

Build a Research Prompt Workflow.

Requirements:

- Accept a research question.
- Create a short plan.
- Generate search queries.
- Summarize findings.
- Produce a final answer with assumptions and missing information.

Folder structure:

```text
research-workflow/
  app/
    main.py
    workflow.py
    prompts.py
    schemas.py
  tests/
    test_workflow_limits.py
```

## 22. Production-Level Project

Build a Bounded Research Agent.

Real-world problem:

- Users need reliable research summaries without uncontrolled tool loops.

Architecture:

```text
[User] -> [FastAPI] -> [Workflow Orchestrator]
                         |
                         +-> [Planner]
                         +-> [Search Tool]
                         +-> [Reader Tool]
                         +-> [State Store]
                         +-> [Final Writer]
```

Tech stack:

- FastAPI
- PostgreSQL for runs and traces
- Redis for active state and limits
- LLM API
- Search or document retrieval tool

Scaling strategy:

- Queue long-running runs.
- Limit max steps and cost.
- Store traces for debugging.
- Cache repeated searches.
- Add human review for important reports.

## Quiz

1. What problem does advanced prompting solve?
2. What is task decomposition?
3. What is ReAct?
4. Why can reasoning-looking text still be wrong?
5. Why do agents need max step limits?
6. What should be logged in a workflow trace?
7. When is advanced prompting overkill?
8. How do tool schemas improve safety?
9. What is reflection or critique prompting?
10. How would you design a bounded research workflow?

## Knowledge Check

You should be able to explain advanced prompting patterns, choose when to use them, and design safe boundaries for multi-step model workflows.

Are you ready for the next section?
