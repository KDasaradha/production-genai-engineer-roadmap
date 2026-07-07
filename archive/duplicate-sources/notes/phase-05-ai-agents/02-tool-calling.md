# Tool Calling

## 1. Problem Statement

Tool calling solves the problem of connecting LLMs to external capabilities such as search, databases, APIs, calculators, file systems, email, calendars, CRM systems, and code execution.

LLMs generate text. Real applications need actions and live data. Tool calling gives the model a controlled way to request backend functions, while the backend decides whether and how to execute them.

Without tool calling:

- models cannot access live/private systems
- agents cannot take useful actions
- users get generated guesses instead of real data
- workflows require manual copy-paste

Without safe tool boundaries:

- models may call dangerous tools
- users may manipulate tool arguments
- unauthorized actions may happen
- auditability is lost

Real-world analogy: the LLM is a planner. Tools are company systems. The planner can request an action, but security and execution must happen through approved backend processes.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Tool calling lets an LLM request predefined functions with structured arguments. |
| Key terminology | tool, function, schema, argument validation, execution, observation, side effect, idempotency |
| Simple explanation | The model asks to use a tool; the backend validates and runs it. |
| Mental model | The model suggests; the backend executes safely. |
| Easy example | The model calls a `search_docs(query)` tool before answering. |
| Use When | The model needs live data, calculations, retrieval, or actions. |
| Avoid When | The answer can be generated or retrieved directly without external action. |
| Advantages | Connects AI to real systems and workflows. |
| Tradeoffs | Security, permissions, validation, and auditing become critical. |
| Limitations | Models can choose wrong tools or provide bad arguments. |
| Production Example | An agent checks customer status in CRM before drafting a response. |
| Interview Answer | Tool calling exposes predefined backend functions to the model through schemas, but the backend validates permissions and executes the tool. |

## 3. Intermediate Explanation

Tool definition usually includes:

- name
- description
- input schema
- output schema
- permission scope
- whether it is read-only or write-capable
- timeout
- retry policy
- audit policy

Tool categories:

| Type | Example | Risk |
| --- | --- | --- |
| Read-only | search docs, lookup ticket | low to medium |
| Computational | calculator, parser | low |
| Retrieval | vector search, database lookup | permission risk |
| Write action | send email, update CRM | high |
| Code execution | run tests, execute script | very high |
| External API | payment, calendar, shipping | side-effect risk |

Data flow:

```text
LLM selects tool -> backend validates schema -> checks permission -> executes tool -> returns observation -> LLM continues
```

## 4. Advanced Explanation

Production tool calling is backend architecture, not prompt text.

Optimization techniques:

- keep tools small and specific
- use strict schemas
- use enums where possible
- make tools read-only by default
- require approval for writes
- validate arguments in code
- enforce auth outside the model
- make write tools idempotent
- log all tool calls
- add timeouts and retries

Performance considerations:

- tool calls add latency
- retries can duplicate side effects if not idempotent
- slow tools can block agent loops
- tool output can consume context tokens

Scaling considerations:

- run slow tools asynchronously
- rate-limit expensive tools
- cache read-only results
- queue write operations
- monitor tool error rate

Production challenges:

- prompt injection causing bad tool calls
- invalid arguments
- missing permissions
- stale tool results
- accidental duplicate writes
- unsafe code execution

## 5. Internal Working

```text
Prompt includes available tools
  |
  v
Model returns tool name and arguments
  |
  v
Backend validates:
  - schema
  - auth
  - permissions
  - policy
  |
  v
Tool executes
  |
  v
Observation returned to model
  |
  v
Model answers or calls another tool
```

Detailed lifecycle:

1. Developer registers approved tools.
2. User asks a task.
3. Model decides a tool is needed.
4. Model emits tool call with arguments.
5. Backend validates arguments.
6. Backend checks user permissions.
7. Tool executes with timeout.
8. Result is logged.
9. Observation is passed back to model.
10. Final answer or next tool call is produced.

## 6. When To Use

Use tool calling for:

- live data lookup
- document search
- calculations
- ticket lookup
- CRM lookup
- calendar scheduling
- email drafting
- database query
- code search
- workflow automation

Ideal use cases:

- agents
- RAG systems
- support assistants
- coding assistants
- workflow automation

## 7. When NOT To Use

Avoid tool calling when:

- a static answer is enough
- deterministic backend code should run directly
- tool side effects are too risky
- you cannot enforce permissions
- you cannot log actions

Better alternatives:

- direct API endpoint
- backend workflow
- RAG-only retrieval
- manual approval
- normal scheduled job

## 8. Advantages

- Gives models live capabilities.
- Reduces hallucination for factual lookups.
- Enables workflow automation.
- Supports agentic systems.
- Makes AI apps more useful than plain chat.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Capability vs risk | Tools make agents useful but can create side effects. |
| Flexibility vs security | More tools mean larger attack surface. |
| Automation vs approval | Automatic writes are fast but risky. |
| Tool output vs context budget | Large results can overflow prompts. |

## 10. Limitations

- The model may choose the wrong tool.
- The model may provide invalid arguments.
- Tool output may be incomplete or stale.
- Tool execution can fail.
- Tool calling does not replace authorization.
- Dangerous tools need sandboxing or approval.

## 11. Real-World Examples

Startup example: a customer support assistant calls `search_help_articles` and `lookup_order_status`.

Enterprise example: an internal agent calls `get_customer_profile`, drafts an email, and waits for employee approval.

FAANG-style example: a coding assistant uses code search, test execution, and issue tracker tools with sandboxing and audit logs.

Production system: a tool gateway exposes typed read-only tools by default, requires approval for writes, and stores every tool call in PostgreSQL.

## 12. Architecture Diagram

```text
[LLM]
  |
  v
[Tool Call Request]
  |
  v
[Tool Gateway]
  |
  +-> [Schema Validator]
  +-> [Auth/Policy Check]
  +-> [Tool Executor]
  +-> [Audit Logger]
  |
  v
[Observation Returned to LLM]
```

## 13. Python Implementation

Tool schema:

```python
from dataclasses import dataclass
from typing import Literal

@dataclass
class ToolDefinition:
    name: str
    description: str
    access_level: Literal["read", "write"]
    requires_approval: bool
```

Tool function:

```python
def calculate_total(price: float, quantity: int) -> float:
    if price < 0 or quantity < 0:
        raise ValueError("price and quantity must be positive")
    return price * quantity
```

Tool permission check:

```python
def can_use_tool(user_roles: set[str], tool: ToolDefinition) -> bool:
    if tool.access_level == "read":
        return "user" in user_roles or "admin" in user_roles
    return "admin" in user_roles
```

Tool call record:

```python
@dataclass
class ToolCall:
    run_id: str
    tool_name: str
    arguments: dict[str, object]
    status: str
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class CalculateRequest(BaseModel):
    price: float = Field(ge=0)
    quantity: int = Field(ge=0)

class ToolResult(BaseModel):
    tool_name: str
    result: float

@app.post("/tools/calculate-total", response_model=ToolResult)
async def calculate_total_tool(request: CalculateRequest) -> ToolResult:
    result = calculate_total(request.price, request.quantity)
    return ToolResult(tool_name="calculate_total", result=result)
```

Production-ready structure:

```text
app/
  api/routes/tools.py
  services/tool_registry.py
  services/tool_executor.py
  services/tool_policy.py
  services/audit_logger.py
  schemas/tool_schemas.py
```

## 15. Database Integration

PostgreSQL:

```text
tools(id, name, access_level, requires_approval, enabled)
tool_calls(id, run_id, user_id, tool_name, arguments_json, status, created_at)
tool_results(id, tool_call_id, result_json, error_message, latency_ms)
approvals(id, tool_call_id, approver_id, status, decided_at)
```

Redis:

- rate limit tool calls
- store temporary approval tokens
- cache read-only tool results

Vector DB:

- search tools often query vector indexes

## 16. Production Considerations

- Allowlist tools.
- Validate all arguments with schemas.
- Enforce permissions in backend code.
- Require approval for side effects.
- Make write tools idempotent.
- Add timeouts.
- Log tool calls and results.
- Redact sensitive arguments.
- Prevent arbitrary code execution.
- Limit tool output size.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Letting the model run arbitrary commands | Expose only allowlisted tools |
| Beginner | Trusting model arguments | Validate with schemas |
| Intermediate | Tool auth in prompt only | Enforce auth in backend code |
| Intermediate | Returning huge tool outputs | Summarize or paginate tool results |
| Production | No audit logs | Store tool calls, results, and approvals |
| Production | Non-idempotent write tools | Add idempotency keys and approval |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is tool calling? | A way for an LLM to request predefined backend functions with structured arguments. |
| Basic | Who executes the tool? | The backend executes the tool after validation and authorization. |
| Intermediate | Why are schemas important? | They constrain and validate tool arguments before execution. |
| Advanced | How do you secure tool-using agents? | Use allowlisted tools, typed schemas, backend permissions, audit logs, rate limits, idempotency, and approval for writes. |
| Scenario | The model wants to send an email. | Draft the email, validate recipients/content, require user approval, then send through a safe backend tool. |

## 19. System Design Discussion

Tool calling is the boundary between language and action. The model can decide that a tool is useful, but backend systems must decide whether the tool call is valid and allowed.

Design decisions:

- read-only vs write tools
- approval policy
- argument schema
- output size limit
- timeout and retry policy
- audit logging
- sandboxing
- idempotency

## 20. Hands-On Assignment

- Easy: Define three read-only tools with names and schemas.
- Medium: Build a tool permission checker.
- Hard: Design an approval flow for write tools.

## 21. Mini Project

Build a Tool Gateway Demo.

Requirements:

- Register tools.
- Validate tool arguments.
- Execute read-only tools.
- Reject unauthorized write tools.
- Log tool calls.

Folder structure:

```text
tool-gateway/
  app/
    main.py
    registry.py
    executor.py
    policy.py
    schemas.py
  tests/
    test_tool_policy.py
```

## 22. Production-Level Project

Build a Secure Agent Tool Gateway.

Real-world problem:

- Agents need tools, but companies need security, approvals, and auditability.

Architecture:

```text
[Agent] -> [Tool Gateway]
             |
             +-> [Schema Validation]
             +-> [Permission Service]
             +-> [Approval Service]
             +-> [Tool Executor]
             +-> [Audit Log]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- Pydantic
- background workers for long tools

Scaling strategy:

- cache read-only results
- queue slow tools
- rate-limit by tenant
- monitor tool error rate
- alert on suspicious tool attempts

## Quiz

1. What is tool calling?
2. Why should tools be predefined?
3. Why validate tool arguments?
4. Why should permissions be enforced outside the prompt?
5. What is a side effect?
6. What is idempotency?
7. When should tool calls require approval?
8. What should be logged for each tool call?
9. Why limit tool output size?
10. How would you secure an email-sending tool?

## Knowledge Check

You should be able to design a safe tool-calling system with schemas, backend authorization, audit logs, idempotency, and human approval for risky actions.

Are you ready for the next section?
