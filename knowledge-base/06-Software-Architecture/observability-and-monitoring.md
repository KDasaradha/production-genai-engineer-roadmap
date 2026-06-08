# Observability and Monitoring

## 1. Problem Statement

Observability and monitoring solve the problem of understanding how an AI system behaves in production.

AI systems fail in ways normal backends do not. The API may be healthy, but retrieval may return bad chunks. The model may answer slowly. Token cost may spike. A prompt version may increase refusals. A user may report hallucinations. Observability helps you see these problems.

Without observability:

- hallucination reports are hard to debug
- token cost can grow unnoticed
- retrieval failures are invisible
- provider outages are confusing
- latency bottlenecks are hard to locate
- production quality cannot improve

Real-world analogy: observability is the dashboard, flight recorder, and warning system for your AI application.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Observability uses logs, metrics, and traces to understand system behavior; AI observability also tracks model, retrieval, cost, and quality signals. |
| Key terminology | logs, metrics, traces, request ID, latency, token usage, cost, retrieval trace, feedback |
| Simple explanation | Record what happened, measure important numbers, and trace requests across services. |
| Mental model | If a user says "AI gave a bad answer," you should be able to replay the path. |
| Easy example | Log model name, prompt version, latency, token count, and retrieved chunks. |
| Use When | Any AI feature is used by real users. |
| Avoid When | Never skip for production; keep it lighter for prototypes. |
| Advantages | Faster debugging, cost control, reliability, quality improvement. |
| Tradeoffs | Telemetry storage, privacy risk, implementation effort. |
| Limitations | Observability shows evidence; engineers still need diagnosis. |
| Production Example | RAG dashboard tracks retrieval quality, citation coverage, cost, latency, and user feedback. |
| Interview Answer | AI observability tracks normal backend health plus model latency, token usage, cost, retrieval results, prompt versions, citations, and answer feedback. |

## 3. Intermediate Explanation

Core observability types:

| Type | What It Answers | Example |
| --- | --- | --- |
| Logs | What happened? | model call failed |
| Metrics | How often/how much? | p95 latency, token cost |
| Traces | Where did time go? | API -> retrieval -> rerank -> LLM |
| Events | What user/system action happened? | feedback submitted |
| Eval reports | Is quality improving? | retrieval recall score |

AI-specific metrics:

- model latency
- prompt tokens
- completion tokens
- total cost
- model provider errors
- retrieval latency
- retrieved chunk scores
- citation coverage
- no-context rate
- refusal rate
- bad feedback rate
- fallback model rate
- agent step count
- tool error rate

Data flow:

```text
Request -> logs + metrics + traces -> dashboards + alerts -> debugging and improvement
```

## 4. Advanced Explanation

AI observability should connect product quality to technical traces.

Optimization techniques:

- use request IDs across all services
- log prompt/model/retrieval versions
- store retrieval trace IDs
- separate user-visible content from sensitive logs
- track cost per tenant and feature
- add dashboards for latency, errors, cost, and quality
- alert on spikes in failures or cost
- connect user feedback to traces

Performance considerations:

- too much logging can increase cost
- logging full prompts can create privacy risk
- tracing every token may be excessive
- metrics cardinality can explode with user IDs or prompt text

Scaling considerations:

- aggregate usage data asynchronously
- sample traces for high-volume endpoints
- store detailed traces for failures
- roll up metrics by tenant, feature, model, and endpoint

Production challenges:

- sensitive data in prompts
- missing request correlation
- no link between feedback and retrieval
- dashboards without actionable alerts
- high-cardinality metrics
- hidden provider errors

## 5. Internal Working

```text
User request
  |
  v
Request ID created
  |
  v
Each service logs with same request ID
  |
  v
Metrics emitted for latency, tokens, cost, errors
  |
  v
Trace links API, retrieval, model, tools
  |
  v
Dashboards and alerts detect issues
  |
  v
Engineers debug using trace and logs
```

Detailed lifecycle:

1. API receives request.
2. Middleware creates request ID.
3. Retrieval logs query, filters, chunks, and scores.
4. Model gateway logs model, tokens, latency, and errors.
5. Response logs citations and status.
6. User feedback links to message and trace.
7. Dashboards aggregate trends.
8. Alerts fire on anomalies.

## 6. When To Use

Use observability for:

- AI chat apps
- RAG systems
- agent workflows
- model gateways
- document ingestion
- local model serving
- production APIs
- evaluation pipelines

## 7. When NOT To Use

Do not log sensitive raw prompts or documents without policy, consent, and redaction.

For prototypes:

- log minimal debug info
- avoid storing sensitive text
- add structured logs early

## 8. Advantages

- Faster incident response.
- Better cost control.
- Easier hallucination debugging.
- Better retrieval improvement.
- Provider outage detection.
- Stronger production credibility.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Detail vs privacy | More logs help debugging but can expose sensitive data. |
| Observability vs cost | Metrics, traces, and logs cost storage and compute. |
| Alerting vs noise | Too many alerts cause fatigue. |
| Sampling vs completeness | Sampling saves cost but may miss rare issues. |

## 10. Limitations

- Observability does not fix issues automatically.
- Feedback can be biased.
- Metrics can hide individual failures.
- Logs may be incomplete if not designed well.
- Quality monitoring is harder than uptime monitoring.

## 11. Real-World Examples

Startup example: track token usage per customer to prevent unexpected bills.

Enterprise example: RAG assistant links every bad-feedback report to retrieved chunks and prompt version.

FAANG-style example: model platform tracks latency, cost, eval scores, safety events, and model routing across products.

Production system: AI backend uses request IDs, OpenTelemetry traces, model usage tables, retrieval logs, and dashboards.

## 12. Architecture Diagram

```text
[FastAPI]
   |
   +-> [Structured Logs]
   +-> [Metrics]
   +-> [Distributed Traces]
   |
   v
[Observability Platform]
   |
   +-> [Dashboards]
   +-> [Alerts]
   +-> [Debug Traces]
```

AI trace:

```text
request -> auth -> retrieval -> rerank -> prompt build -> model call -> response -> feedback
```

## 13. Python Implementation

Structured log fields:

```python
from dataclasses import dataclass

@dataclass
class ModelCallLog:
    request_id: str
    model: str
    prompt_version: str
    prompt_tokens: int
    completion_tokens: int
    latency_ms: int
    status: str
```

Cost calculation:

```python
def estimate_cost(total_tokens: int, cost_per_1k_tokens: float) -> float:
    return (total_tokens / 1000) * cost_per_1k_tokens
```

Retrieval trace:

```python
@dataclass
class RetrievedChunkLog:
    request_id: str
    chunk_id: str
    score: float
    rank: int
    source: str
```

## 14. FastAPI Implementation

```python
from time import perf_counter
from uuid import uuid4
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_request_id_and_timing(request: Request, call_next):
    request_id = request.headers.get("x-request-id", str(uuid4()))
    start = perf_counter()
    response = await call_next(request)
    latency_ms = int((perf_counter() - start) * 1000)
    response.headers["x-request-id"] = request_id
    response.headers["x-latency-ms"] = str(latency_ms)
    return response
```

Production-ready structure:

```text
app/
  middleware/request_context.py
  services/metrics_service.py
  services/usage_logger.py
  services/retrieval_trace_logger.py
  repositories/usage_repository.py
  repositories/feedback_repository.py
```

## 15. Database Integration

PostgreSQL:

```text
model_usage(id, request_id, user_id, model, prompt_tokens, completion_tokens, cost, latency_ms)
retrieval_traces(id, request_id, query, filters_json, latency_ms)
retrieval_trace_chunks(id, trace_id, chunk_id, score, rank, source)
answer_feedback(id, request_id, message_id, rating, reason, created_at)
security_events(id, request_id, user_id, event_type, risk_level)
```

Metrics backend:

- request rate
- error rate
- p95 latency
- token usage
- cost per tenant
- provider failures

## 16. Production Considerations

- Add request IDs.
- Use structured logs.
- Redact sensitive data.
- Track cost per tenant.
- Track model and prompt versions.
- Track retrieval traces.
- Link user feedback to traces.
- Alert on cost spikes.
- Alert on provider failures.
- Monitor no-context and bad-feedback rates.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Using only `print()` | Use structured logs |
| Beginner | No request IDs | Add correlation IDs |
| Intermediate | Logging full sensitive prompts | Redact or store references |
| Intermediate | Only monitoring API uptime | Monitor model, retrieval, cost, and quality |
| Production | No feedback-to-trace link | Store request/message IDs with feedback |
| Production | High-cardinality metrics | Aggregate by feature, tenant, model, endpoint |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is observability? | Logs, metrics, and traces that help understand system behavior. |
| Basic | What should AI systems monitor? | Latency, errors, token usage, cost, retrieval quality, model versions, citations, and feedback. |
| Intermediate | Why use request IDs? | To correlate logs and traces across services for one user request. |
| Advanced | How debug a bad RAG answer? | Inspect the request trace, retrieved chunks, scores, filters, prompt version, model output, citations, and feedback. |
| Scenario | Cost spikes overnight. | Break down token usage by tenant, endpoint, model, prompt version, and deployment. |

## 19. System Design Discussion

AI observability must cover:

- system reliability
- model behavior
- retrieval quality
- cost
- safety
- product feedback

Design decisions:

- what to log
- what to redact
- what metrics to alert on
- trace sampling
- feedback schema
- retention policy
- dashboards by stakeholder

## 20. Hands-On Assignment

- Easy: Add request ID middleware.
- Medium: Design model usage log schema.
- Hard: Build a retrieval trace schema linked to answer feedback.

## 21. Mini Project

Build AI Usage Logging.

Requirements:

- Log model name.
- Log prompt and completion tokens.
- Log latency.
- Log cost estimate.
- Link logs to request ID.

Folder structure:

```text
ai-usage-logging/
  app/
    main.py
    middleware.py
    usage_logger.py
    schemas.py
  tests/
    test_usage_cost.py
```

## 22. Production-Level Project

Build Observability for a RAG Platform.

Real-world problem:

- Team needs to debug bad answers, track cost, and monitor production health.

Architecture:

```text
[AI API] -> [Logs/Metrics/Traces]
         -> [Usage DB]
         -> [Retrieval Trace DB]
         -> [Feedback DB]
         -> [Dashboards + Alerts]
```

Tech stack:

- FastAPI
- PostgreSQL
- metrics/tracing backend
- logging pipeline
- dashboard tool

Scaling strategy:

- aggregate usage asynchronously
- sample successful traces
- store detailed traces for failures
- set alerts for cost, latency, errors, and bad feedback

## Quiz

1. What are logs?
2. What are metrics?
3. What are traces?
4. Why do AI systems need token usage monitoring?
5. What is a retrieval trace?
6. Why link feedback to request IDs?
7. What privacy risk exists in AI logs?
8. What is p95 latency?
9. What alerts matter for AI systems?
10. How would you debug a hallucinated RAG answer?

## Knowledge Check

You should be able to design observability for AI apps covering backend health, model usage, retrieval traces, cost, safety, and user feedback.

Are you ready for the next section?
