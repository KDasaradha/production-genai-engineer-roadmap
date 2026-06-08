# Quantization and Distillation

## 1. Problem Statement

Quantization and distillation solve the problem of making models cheaper, smaller, faster, or easier to deploy.

Large models can be powerful but expensive. They need memory, compute, GPUs, and careful serving. Production systems often need lower latency and lower cost. Compression techniques help, but they can reduce quality if used carelessly.

Without optimization:

- inference cost may be too high
- latency may be too slow
- models may not fit available hardware
- local deployment may be impractical
- high-volume tasks may be uneconomical

Real-world analogy: quantization is like storing high-resolution data in a smaller format. Distillation is like training a junior specialist to imitate a senior expert for a narrow set of tasks.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Quantization reduces model numeric precision; distillation trains a smaller model to imitate a larger teacher model. |
| Key terminology | precision, FP32, FP16, INT8, INT4, teacher model, student model, latency, throughput |
| Simple explanation | Make the model smaller or teach a smaller model to behave like a bigger one. |
| Mental model | Trade some quality for speed, memory, and cost improvements. |
| Easy example | Run a quantized local model that fits on a laptop GPU. |
| Use When | Model serving is too slow, expensive, or memory-heavy. |
| Avoid When | Quality loss is unacceptable and cost is manageable. |
| Advantages | Lower memory, lower cost, faster serving, local deployment. |
| Tradeoffs | Possible quality drop, more evaluation, serving complexity. |
| Limitations | Compression may hurt reasoning, rare cases, or domain accuracy. |
| Production Example | Distill a high-volume support classifier into a smaller model. |
| Interview Answer | Quantization and distillation optimize inference cost and latency, but must be evaluated because they can reduce quality. |

## 3. Intermediate Explanation

Quantization:

| Precision | Meaning | Use |
| --- | --- | --- |
| FP32 | full precision float | training, highest precision |
| FP16/BF16 | half precision | common GPU inference/training |
| INT8 | 8-bit integer | faster/lower memory inference |
| INT4 | 4-bit integer | very low memory local models |

Distillation:

```text
Large teacher model -> generates labels/explanations -> smaller student model learns pattern
```

Common goals:

- reduce memory
- increase tokens per second
- reduce cost per request
- fit model on available hardware
- serve high-volume narrow tasks

Data flow:

```text
Base model -> compression/teacher-student training -> benchmark -> evaluation -> deployment
```

## 4. Advanced Explanation

Optimization should be driven by measured bottlenecks. Do not quantize or distill because it sounds advanced. Do it because latency, memory, or cost data proves the need.

Optimization techniques:

- benchmark baseline model first
- test multiple precision levels
- evaluate on task-specific data
- route simple tasks to optimized models
- keep larger model fallback for hard cases
- monitor quality after deployment
- combine quantization with batching
- use distillation for high-volume narrow tasks

Performance considerations:

- quantization can improve memory and throughput
- some hardware handles certain precisions better
- smaller models may be faster but less capable
- distillation requires teacher output generation

Scaling considerations:

- use model gateway for routing
- track cost by model
- autoscale based on queue depth and GPU utilization
- keep rollback model ready
- benchmark under realistic traffic

Production challenges:

- quality regression on edge cases
- hardware-specific performance differences
- model loading time
- inaccurate benchmarks
- user-visible degradation after optimization

## 5. Internal Working

Quantization lifecycle:

```text
Original model weights
  |
  v
Convert to lower precision
  |
  v
Load optimized model
  |
  v
Benchmark latency/memory
  |
  v
Evaluate quality
  |
  v
Deploy or reject
```

Distillation lifecycle:

```text
Teacher model
  |
  v
Generate training labels or responses
  |
  v
Train smaller student model
  |
  v
Evaluate against teacher and ground truth
  |
  v
Deploy for narrow task
```

## 6. When To Use

Use quantization when:

- model does not fit memory
- inference is too expensive
- local deployment is needed
- GPU memory is limited
- throughput needs improvement

Use distillation when:

- one task has high request volume
- a large model performs well but is expensive
- a smaller model can learn the task
- latency matters

## 7. When NOT To Use

Avoid compression when:

- quality is already fragile
- request volume is low
- hosted model cost is acceptable
- you cannot evaluate regressions
- task requires strongest reasoning available

Better alternatives:

- prompt reduction
- caching
- model routing
- RAG improvements
- batching
- provider selection

## 8. Advantages

- Lower memory usage.
- Faster inference.
- Lower cost.
- Enables local deployment.
- Better throughput.
- Can make high-volume features economical.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Cost vs quality | Smaller/quantized models may perform worse. |
| Speed vs reasoning | Faster models may be weaker on complex tasks. |
| Optimization vs maintenance | More model variants require management. |
| Local control vs ops burden | Self-hosting optimized models needs infrastructure. |

## 10. Limitations

- Compression can hurt edge cases.
- Distillation quality depends on teacher and data.
- Quantization performance depends on hardware/runtime.
- Smaller models may fail complex reasoning.
- Evaluation is mandatory.

## 11. Real-World Examples

Startup example: quantize a local assistant to run on cheaper hardware.

Enterprise example: distill a ticket classifier from a large model into a smaller internal model.

FAANG-style example: model routing sends simple tasks to small optimized models and complex tasks to larger models.

Production system: a model gateway compares latency, cost, and quality for FP16, INT8, and INT4 variants before rollout.

## 12. Architecture Diagram

```text
[Baseline Model]
      |
      +-> [Quantized Variant] -> [Benchmark]
      |
      +-> [Distilled Student] -> [Evaluation]
                               -> [Model Registry]
                               -> [Model Gateway]
```

## 13. Python Implementation

Benchmark record:

```python
from dataclasses import dataclass

@dataclass
class ModelBenchmark:
    model_name: str
    precision: str
    memory_gb: float
    latency_ms: int
    tokens_per_second: float
    quality_score: float
```

Cost comparison:

```python
def monthly_inference_cost(cost_per_request: float, monthly_requests: int) -> float:
    return cost_per_request * monthly_requests
```

Model selection:

```python
def choose_model(benchmarks: list[ModelBenchmark], min_quality: float) -> ModelBenchmark:
    viable = [item for item in benchmarks if item.quality_score >= min_quality]
    if not viable:
        raise ValueError("no model meets quality threshold")
    return min(viable, key=lambda item: item.latency_ms)
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class BenchmarkRequest(BaseModel):
    model_name: str
    precision: str
    latency_ms: int = Field(ge=0)
    memory_gb: float = Field(ge=0)
    quality_score: float = Field(ge=0, le=1)

@app.post("/models/benchmarks")
async def record_benchmark(request: BenchmarkRequest) -> dict[str, str]:
    if request.quality_score < 0.7:
        raise HTTPException(status_code=422, detail="quality score too low for deployment candidate")
    return {"status": "candidate_recorded", "model": request.model_name}
```

Production-ready structure:

```text
app/
  api/routes/model_benchmarks.py
  services/benchmark_service.py
  services/model_router.py
  services/model_registry.py
  repositories/benchmark_repository.py
```

## 15. Database Integration

PostgreSQL:

```text
model_variants(id, base_model, variant_name, precision, runtime, status)
model_benchmarks(id, variant_id, latency_ms, tokens_per_second, memory_gb, quality_score)
model_routing_rules(id, task_type, variant_id, min_quality, max_latency_ms)
```

Redis:

- cache routing decisions
- store live model health
- track queue depth

Monitoring:

- latency
- tokens/sec
- GPU memory
- quality feedback
- fallback rate
- cost per task

## 16. Production Considerations

- Benchmark before and after optimization.
- Evaluate task-specific quality.
- Keep baseline model for comparison.
- Use canary rollout.
- Monitor fallback rate.
- Track hardware/runtime details.
- Store model variant versions.
- Route complex tasks to stronger models.
- Roll back on quality regression.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Assuming smaller is always better | Measure quality and latency |
| Beginner | Only comparing model size | Compare task quality and cost |
| Intermediate | No task-specific evals | Use real examples |
| Intermediate | Ignoring hardware | Benchmark on target hardware |
| Production | No rollback | Keep previous model available |
| Production | No online monitoring | Track real feedback and failures |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is quantization? | Reducing numeric precision of model weights to lower memory and improve inference efficiency. |
| Basic | What is distillation? | Training a smaller student model to imitate a larger teacher model. |
| Intermediate | What is the main tradeoff? | Lower cost and latency may reduce quality. |
| Advanced | How do you deploy an optimized model safely? | Benchmark baseline, evaluate task quality, canary rollout, monitor regressions, and keep rollback. |
| Scenario | Local model is too slow. | Try quantization, smaller model, batching, prompt reduction, vLLM, or route simple tasks to optimized variants. |

## 19. System Design Discussion

Quantization and distillation fit inside model operations and routing.

Design decisions:

- optimize or use hosted API
- quantize or distill
- precision level
- target hardware
- quality threshold
- routing rules
- fallback model
- monitoring metrics

## 20. Hands-On Assignment

- Easy: Compare cost of two model variants.
- Medium: Create a benchmark table with latency, memory, and quality.
- Hard: Design model routing rules based on task complexity and quality threshold.

## 21. Mini Project

Build a Model Benchmark Tracker.

Requirements:

- Record model variant.
- Record precision.
- Record latency, memory, and quality score.
- Choose fastest model above quality threshold.
- Print cost comparison.

Folder structure:

```text
model-benchmark-tracker/
  app/
    main.py
    benchmarks.py
    router.py
    schemas.py
  tests/
    test_router.py
```

## 22. Production-Level Project

Build a Cost-Aware Model Router.

Real-world problem:

- A company wants to reduce AI serving cost without hurting quality.

Architecture:

```text
[AI Request] -> [Task Classifier]
             -> [Model Router]
             -> [Optimized Model or Large Model]
             -> [Monitoring + Feedback]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- local or hosted models
- monitoring dashboard

Scaling strategy:

- route simple tasks to optimized models
- keep fallback to larger models
- track quality by task type
- canary new model variants
- monitor cost savings and regressions

## Quiz

1. What is quantization?
2. What is distillation?
3. What is a teacher model?
4. What is a student model?
5. Why does precision affect memory?
6. What is the risk of INT4 quantization?
7. Why evaluate task-specific quality?
8. What is model routing?
9. Why keep a fallback model?
10. How would you reduce model serving cost safely?

## Knowledge Check

You should be able to explain quantization, distillation, model benchmarking, and safe cost-aware deployment.

Are you ready for the next section?
