# GPU Deployment and Cost Optimization

## 1. Problem Statement

GPU deployment and cost optimization solve the problem of serving AI workloads efficiently without wasting expensive compute or overspending on model calls.

AI systems can become expensive quickly. Hosted APIs charge by token. Local models need GPUs. Long prompts increase latency and cost. High-volume features can create large bills. Production AI engineers must understand cost and performance as first-class design constraints.

Without cost optimization:

- token bills grow unexpectedly
- GPUs sit idle while costing money
- users experience slow responses
- simple tasks use expensive models
- scaling becomes uneconomical

Real-world analogy: model serving is like running a delivery fleet. You need the right vehicle for each job, route optimization, fuel tracking, and maintenance.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | GPU deployment runs model workloads on GPU infrastructure; cost optimization reduces unnecessary model, token, storage, and compute spend. |
| Key terminology | GPU, utilization, VRAM, batching, throughput, tokens/sec, queue depth, autoscaling, model routing |
| Simple explanation | Run models efficiently and choose cheaper paths when quality is still good. |
| Mental model | Use the smallest reliable model and infrastructure for the task. |
| Easy example | Route simple classification to a small model and complex reasoning to a stronger model. |
| Use When | Serving local/open models or managing production AI costs. |
| Avoid When | Hosted APIs are simpler, cheaper, and compliant for your volume. |
| Advantages | Lower cost, better throughput, more control, improved scalability. |
| Tradeoffs | Infrastructure complexity and quality tradeoffs. |
| Limitations | Optimization can reduce quality if poorly measured. |
| Production Example | vLLM serves local models with batching while model gateway tracks cost per tenant. |
| Interview Answer | GPU and cost optimization balance model quality, latency, throughput, memory, batching, routing, caching, and utilization. |

## 3. Intermediate Explanation

Important metrics:

| Metric | Meaning |
| --- | --- |
| GPU utilization | how busy the GPU is |
| VRAM usage | GPU memory used |
| tokens/sec | generation throughput |
| queue depth | waiting requests |
| p95 latency | slow-user experience |
| cost/request | average request cost |
| cost/tenant | tenant-level spend |
| fallback rate | how often primary model fails/routes away |
| cache hit rate | how often cache avoids work |

Cost levers:

- model size
- prompt length
- output length
- model routing
- caching
- batching
- quantization
- RAG top-k
- reranking usage
- provider choice
- local vs hosted serving

Data flow:

```text
Request -> task classification -> model routing -> inference -> usage logging -> cost dashboard
```

## 4. Advanced Explanation

Cost optimization should never be blind. You need baseline metrics, quality thresholds, and rollback.

Optimization techniques:

- reduce prompt boilerplate
- summarize or trim chat history
- cache embeddings
- cache stable responses when safe
- use smaller models for simple tasks
- route complex tasks to stronger models
- batch local model requests
- quantize local models
- stream responses
- set max output tokens
- monitor cost by feature

Performance considerations:

- batching improves throughput but can increase per-request wait
- long context increases memory and latency
- local model serving needs warm replicas
- GPU underutilization wastes money
- overutilization increases queue time

Scaling considerations:

- autoscale by queue depth or GPU utilization
- separate model-serving workloads from API workloads
- use model gateway for routing and fallback
- track tenant budgets
- forecast cost before large launches

Production challenges:

- cost spikes from prompt bugs
- runaway agent loops
- users abusing long prompts
- GPU cold starts
- provider rate limits
- local model quality gaps
- hidden cost from eval jobs

## 5. Internal Working

```text
Request arrives
  |
  v
Token budget and rate limit checked
  |
  v
Task type classified
  |
  v
Model router selects provider/model
  |
  v
Inference runs on hosted API or GPU server
  |
  v
Usage and latency logged
  |
  v
Cost dashboard and alerts updated
```

GPU serving lifecycle:

```text
Model loaded into GPU memory
  |
  v
Requests enter queue
  |
  v
Batch scheduler groups requests
  |
  v
GPU generates tokens
  |
  v
Responses stream back
  |
  v
Metrics update
```

## 6. When To Use

Use GPU deployment when:

- hosting local/open models
- running fine-tuned models
- high volume makes hosted APIs expensive
- data locality is required
- latency improves with local serving

Use cost optimization when:

- every production AI system
- token usage grows
- multiple models are available
- tenants have budgets
- agent workflows call many tools/models

## 7. When NOT To Use

Avoid self-hosted GPU deployment when:

- hosted APIs are cheaper at your volume
- your team cannot operate GPUs
- model quality is not good enough
- workload is small or unpredictable

Better alternatives:

- hosted APIs
- managed model endpoints
- smaller hosted models
- prompt optimization
- caching
- RAG improvements

## 8. Advantages

- Lower cost at scale.
- Better resource control.
- More deployment flexibility.
- Data locality.
- Better throughput with optimized serving.
- Cost visibility by tenant and feature.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Cost vs quality | Cheaper models may answer worse. |
| Throughput vs latency | Batching improves throughput but can add waiting. |
| Local control vs operations | GPUs need monitoring, scaling, and maintenance. |
| Optimization vs complexity | More routing/caching rules require testing. |

## 10. Limitations

- GPUs are expensive.
- GPU capacity may be limited.
- Local models may underperform hosted frontier models.
- Quantization can reduce quality.
- Cost dashboards need accurate usage data.
- Caching is unsafe for personalized sensitive answers unless carefully scoped.

## 11. Real-World Examples

Startup example: reduce monthly LLM spend by trimming prompts and caching embeddings.

Enterprise example: route sensitive internal tasks to local models and complex reasoning to approved hosted models.

FAANG-style example: a model platform routes requests across model tiers and tracks cost, latency, quality, and fallback rate.

Production system: vLLM inference servers run quantized models, while a FastAPI model gateway logs token usage and applies tenant budgets.

## 12. Architecture Diagram

```text
[AI Request]
    |
    v
[Rate Limit + Token Budget]
    |
    v
[Model Router]
    |
    +-> [Small Hosted Model]
    +-> [Large Hosted Model]
    +-> [Local GPU Model Server]
    |
    v
[Usage Logger + Cost Dashboard]
```

GPU serving:

```text
[Queue] -> [Batch Scheduler] -> [GPU Model Server] -> [Streaming Response]
                    |
                    v
              [GPU Metrics]
```

## 13. Python Implementation

Cost estimate:

```python
def token_cost(prompt_tokens: int, completion_tokens: int, input_cost: float, output_cost: float) -> float:
    return (prompt_tokens / 1000) * input_cost + (completion_tokens / 1000) * output_cost
```

Model route:

```python
def route_model(task_type: str, privacy_required: bool, complexity: str) -> str:
    if privacy_required:
        return "local-private-model"
    if task_type == "classification":
        return "small-cheap-model"
    if complexity == "high":
        return "large-reasoning-model"
    return "default-model"
```

Budget check:

```python
def within_budget(current_month_cost: float, estimated_request_cost: float, monthly_budget: float) -> bool:
    return current_month_cost + estimated_request_cost <= monthly_budget
```

GPU metric record:

```python
from dataclasses import dataclass

@dataclass
class GpuMetrics:
    gpu_id: str
    utilization_percent: float
    memory_used_gb: float
    queue_depth: int
    tokens_per_second: float
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class CostEstimateRequest(BaseModel):
    prompt_tokens: int = Field(ge=0)
    completion_tokens: int = Field(ge=0)
    monthly_spend: float = Field(ge=0)
    monthly_budget: float = Field(gt=0)

class CostEstimateResponse(BaseModel):
    estimated_cost: float
    allowed: bool

@app.post("/cost/estimate", response_model=CostEstimateResponse)
async def estimate_request_cost(request: CostEstimateRequest) -> CostEstimateResponse:
    estimated = token_cost(
        request.prompt_tokens,
        request.completion_tokens,
        input_cost=0.01,
        output_cost=0.03,
    )
    allowed = within_budget(request.monthly_spend, estimated, request.monthly_budget)
    if not allowed:
        raise HTTPException(status_code=429, detail="monthly AI budget exceeded")
    return CostEstimateResponse(estimated_cost=estimated, allowed=True)
```

Production-ready structure:

```text
app/
  services/model_router.py
  services/cost_service.py
  services/gpu_metrics_service.py
  services/budget_service.py
  repositories/model_usage_repository.py
  repositories/tenant_budget_repository.py
```

## 15. Database Integration

PostgreSQL:

```text
tenant_budgets(id, tenant_id, monthly_budget, current_spend, reset_at)
model_usage(id, tenant_id, model, prompt_tokens, completion_tokens, cost, latency_ms)
model_routes(id, task_type, selected_model, reason, created_at)
gpu_metrics(id, server_id, utilization, memory_used, queue_depth, tokens_per_second, created_at)
```

Redis:

- live budget counters
- rate limits
- routing cache
- GPU health cache

Monitoring:

- spend per tenant
- spend per feature
- GPU utilization
- queue depth
- p95 latency
- fallback rate
- cache hit rate

## 16. Production Considerations

- Track cost for every model call.
- Enforce tenant budgets.
- Add rate limits for expensive features.
- Monitor GPU utilization.
- Monitor queue depth.
- Route simple tasks to cheaper models.
- Use fallback for local model failures.
- Set max output tokens.
- Alert on cost spikes.
- Evaluate quality before cheaper routing.
- Watch agent loops because they multiply cost.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Ignoring token cost | Track usage from day one |
| Beginner | Using largest model for every task | Route by task complexity |
| Intermediate | No tenant budget | Enforce quotas and alerts |
| Intermediate | Caching personalized answers unsafely | Scope cache by user/tenant and sensitivity |
| Production | GPU underutilization | Batch requests and right-size replicas |
| Production | No quality check after optimization | Use evals and canaries |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | Why are GPUs used for LLMs? | GPUs accelerate large matrix operations used in model inference and training. |
| Basic | What is token cost? | The price paid based on prompt and completion tokens processed by a model. |
| Intermediate | What is model routing? | Choosing a model/provider based on task, cost, latency, privacy, or quality needs. |
| Advanced | How reduce AI system cost safely? | Track usage, reduce prompt tokens, cache safely, route to smaller models, batch local inference, set budgets, and evaluate quality. |
| Scenario | GPU cost is high but latency is still bad. | Check utilization, queue depth, batching, model size, quantization, prompt length, autoscaling, and routing. |

## 19. System Design Discussion

Cost optimization is not only an infrastructure task. It affects:

- prompt design
- model choice
- retrieval top-k
- caching
- streaming
- agent step limits
- tenant quotas
- product pricing

Design decisions:

- hosted vs local
- model tiers
- budget policy
- routing rules
- cache policy
- GPU autoscaling
- quality thresholds
- alerting strategy

## 20. Hands-On Assignment

- Easy: Calculate cost for prompt and completion tokens.
- Medium: Design model routing rules for classification, RAG, and reasoning.
- Hard: Design tenant budget enforcement with Redis counters and PostgreSQL records.

## 21. Mini Project

Build an AI Cost Calculator.

Requirements:

- Accept prompt and completion token counts.
- Estimate cost by model.
- Track monthly usage.
- Reject requests above budget.
- Show cost by feature.

Folder structure:

```text
ai-cost-calculator/
  app/
    main.py
    cost.py
    budgets.py
    schemas.py
  tests/
    test_cost.py
```

## 22. Production-Level Project

Build a Cost-Aware Model Gateway.

Real-world problem:

- A company wants to control AI spend while preserving quality.

Architecture:

```text
[Request] -> [Budget Check]
          -> [Task Classifier]
          -> [Model Router]
          -> [Hosted or Local Model]
          -> [Usage Logger]
          -> [Cost Dashboard]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- local model server or hosted models
- monitoring dashboard

Scaling strategy:

- route by task complexity
- use budgets and rate limits
- cache safe repeated work
- batch local inference
- monitor GPU utilization
- canary cheaper models
- alert on cost anomalies

## Quiz

1. Why are GPUs important for LLM serving?
2. What is GPU utilization?
3. What is VRAM?
4. What is tokens per second?
5. What is model routing?
6. How does prompt length affect cost?
7. Why use tenant budgets?
8. When is local serving cheaper than hosted APIs?
9. What is the risk of aggressive cost optimization?
10. How would you design a cost-aware model gateway?

## Knowledge Check

You should be able to design cost controls for AI systems, explain GPU serving metrics, and build model routing strategies that balance quality, latency, privacy, and spend.

Are you ready for the next section?
