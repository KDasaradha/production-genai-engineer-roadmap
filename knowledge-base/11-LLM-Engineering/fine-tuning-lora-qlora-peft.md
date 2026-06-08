# Fine-Tuning, LoRA, QLoRA, and PEFT

## 1. Problem Statement

Fine-tuning solves the problem of adapting a model's behavior, format, tone, or task performance when prompting and RAG are not enough.

Many beginners think fine-tuning is how you "add knowledge" to a model. That is usually the wrong mental model. If your knowledge changes often, RAG is usually better. Fine-tuning is more useful when you want the model to behave differently or perform a repeated task more consistently.

Without fine-tuning:

- some tasks need very long prompts
- output style may stay inconsistent
- small domain tasks may be expensive with large models
- repeated extraction/classification may remain unreliable
- model behavior may not match business expectations

Real-world analogy: RAG is giving an employee a reference manual. Fine-tuning is training the employee to follow your company's way of working.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Fine-tuning is additional training on task-specific examples. LoRA, QLoRA, and PEFT are efficient ways to adapt models with fewer trainable parameters. |
| Key terminology | dataset, base model, adapter, LoRA, QLoRA, PEFT, training set, validation set, overfitting |
| Simple explanation | You show the model many examples of how inputs should map to outputs. |
| Mental model | Teach behavior and patterns, not frequently changing facts. |
| Easy example | Train a model to always return customer support answers in a specific company tone and format. |
| Use When | You need consistent style, format, classification, extraction, or domain task behavior. |
| Avoid When | You only need fresh factual knowledge from documents. |
| Advantages | Better consistency, shorter prompts, possible cost savings for repeated tasks. |
| Tradeoffs | Requires high-quality data, training, evaluation, and deployment management. |
| Limitations | Can overfit, learn bad examples, or become stale. |
| Production Example | Fine-tuned support assistant that follows a strict answer rubric while RAG provides current policies. |
| Interview Answer | Fine-tuning adapts model behavior using examples; RAG is usually better for dynamic knowledge, while fine-tuning is better for repeated behavior or format. |

## 3. Intermediate Explanation

Fine-tuning approaches:

| Approach | Meaning | Best For | Tradeoff |
| --- | --- | --- | --- |
| Full fine-tuning | update many or all model weights | deep adaptation | expensive and risky |
| LoRA | train small low-rank adapter weights | efficient adaptation | adapter management |
| QLoRA | LoRA with quantized base model | low-memory fine-tuning | more technical complexity |
| PEFT | parameter-efficient fine-tuning family | cheaper adaptation | may be less powerful than full tuning |
| Instruction tuning | examples of instructions and responses | assistant behavior | needs curated data |
| Preference tuning | train from ranked preferences | alignment and style | needs preference data |

Fine-tuning data usually has:

- input
- expected output
- task instructions
- metadata
- quality label
- train/validation split

Data flow:

```text
Examples -> dataset cleaning -> train/validation split -> fine-tuning job -> evaluation -> deployment
```

## 4. Advanced Explanation

Fine-tuning should come after simpler baselines:

1. Try prompt engineering.
2. Try structured outputs.
3. Try RAG for knowledge.
4. Evaluate failures.
5. Fine-tune only when behavior remains consistently weak.

Optimization techniques:

- start with small high-quality datasets
- remove duplicates and contradictions
- keep validation data separate
- include edge cases
- track dataset version
- compare against prompt-only baseline
- evaluate on real production-like examples
- combine fine-tuning with RAG when facts matter

Performance considerations:

- fine-tuned smaller models can be cheaper at scale
- training costs occur upfront
- inference may be faster if prompts become shorter
- quality can drop outside the training distribution

Scaling considerations:

- model registry for versions
- dataset registry for training data
- evaluation pipeline before deployment
- rollback to previous model
- monitoring for drift

Production challenges:

- poor labels
- data leakage
- overfitting
- outdated examples
- unsafe learned behavior
- difficult rollback if not versioned
- evaluation gaps

## 5. Internal Working

```text
Base model
  |
  v
Training examples
  |
  v
Fine-tuning updates model behavior or adapter weights
  |
  v
Validation checks quality and regressions
  |
  v
Model or adapter is deployed
  |
  v
Production monitoring tracks performance
```

Detailed lifecycle:

1. Define the target behavior.
2. Build a prompt-only baseline.
3. Collect examples.
4. Clean and label examples.
5. Split train and validation data.
6. Train model or adapter.
7. Evaluate against baseline.
8. Deploy behind model gateway.
9. Monitor quality, latency, and cost.
10. Roll back if regressions appear.

## 6. When To Use

Use fine-tuning when:

- output format must be very consistent
- domain-specific task patterns repeat
- prompt examples become too long
- classification or extraction needs better accuracy
- style/tone must match brand
- small model needs to perform a narrow task

Ideal use cases:

- support response style
- resume scoring rubric
- contract clause classification
- medical note formatting
- internal coding style
- domain-specific extraction

## 7. When NOT To Use

Avoid fine-tuning when:

- you need updated facts
- documents change frequently
- you have no high-quality examples
- business rules are deterministic
- prompt engineering already works
- legal or safety review is not ready

Better alternatives:

- RAG for knowledge
- prompt engineering for task control
- structured outputs for JSON
- rules engine for deterministic logic
- tool calling for live data

## 8. Advantages

- Improves repeated task consistency.
- Can reduce prompt length.
- Can improve domain behavior.
- Can lower inference cost with smaller models.
- Can enforce tone or format better than prompts alone.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Quality vs data effort | Fine-tuning needs strong examples. |
| Behavior vs knowledge | It teaches patterns better than changing facts. |
| Cost now vs cost later | Training costs upfront may reduce inference cost later. |
| Flexibility vs maintenance | Fine-tuned models need versioning and monitoring. |

## 10. Limitations

- Not ideal for frequently changing knowledge.
- Can overfit training examples.
- Can learn bad labels.
- Needs evaluation and rollback.
- Can be expensive to experiment blindly.
- May reduce general ability on unrelated tasks.

## 11. Real-World Examples

Startup example: fine-tune a small model to classify inbound support tickets cheaply.

Enterprise example: fine-tune a model to produce internal report summaries in a required executive format.

FAANG-style example: train task-specific adapters for high-volume internal workflows while routing complex tasks to larger models.

Production system: RAG retrieves current policy, while a fine-tuned model formats the answer in company support style.

## 12. Architecture Diagram

```text
Training:
[Examples] -> [Cleaning] -> [Fine-Tuning Job] -> [Evaluation] -> [Model Registry]

Serving:
[App] -> [Model Gateway] -> [Fine-Tuned Model]
                         -> [RAG Retriever if facts needed]
```

## 13. Python Implementation

Training example shape:

```python
from dataclasses import dataclass

@dataclass
class TrainingExample:
    instruction: str
    input_text: str
    expected_output: str
    source: str
```

Dataset split:

```python
def split_dataset(examples: list[TrainingExample], validation_ratio: float = 0.2):
    split_index = int(len(examples) * (1 - validation_ratio))
    return examples[:split_index], examples[split_index:]
```

Dataset quality check:

```python
def validate_examples(examples: list[TrainingExample]) -> list[str]:
    errors: list[str] = []
    for index, example in enumerate(examples):
        if not example.expected_output.strip():
            errors.append(f"example {index} has empty output")
        if example.input_text.strip() == example.expected_output.strip():
            errors.append(f"example {index} may be a copy task")
    return errors
```

Model registry record:

```python
@dataclass
class FineTunedModelVersion:
    model_id: str
    base_model: str
    dataset_version: str
    eval_score: float
    status: str
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class FineTuneDatasetRequest(BaseModel):
    name: str = Field(min_length=1)
    examples: list[dict[str, str]]

class DatasetValidationResponse(BaseModel):
    example_count: int
    errors: list[str]
    ready_for_training: bool

@app.post("/fine-tuning/datasets/validate", response_model=DatasetValidationResponse)
async def validate_dataset(request: FineTuneDatasetRequest) -> DatasetValidationResponse:
    examples = [
        TrainingExample(
            instruction=item.get("instruction", ""),
            input_text=item.get("input_text", ""),
            expected_output=item.get("expected_output", ""),
            source=item.get("source", "manual"),
        )
        for item in request.examples
    ]
    errors = validate_examples(examples)
    return DatasetValidationResponse(
        example_count=len(examples),
        errors=errors,
        ready_for_training=len(errors) == 0 and len(examples) >= 20,
    )
```

Production-ready structure:

```text
app/
  api/routes/fine_tuning.py
  services/dataset_service.py
  services/training_service.py
  services/model_registry.py
  services/evaluation_service.py
  repositories/model_repository.py
```

## 15. Database Integration

PostgreSQL:

```text
training_datasets(id, name, version, status, created_at)
training_examples(id, dataset_id, instruction, input_text, expected_output, quality_score)
fine_tuning_jobs(id, dataset_id, base_model, status, started_at, finished_at)
model_versions(id, job_id, model_name, eval_score, status, deployed_at)
```

Redis:

- training job status cache
- rate limit training requests
- queue status for long-running jobs

Object storage:

- dataset files
- training artifacts
- evaluation reports

## 16. Production Considerations

- Version datasets.
- Keep validation sets separate.
- Compare against prompt-only baseline.
- Evaluate before deployment.
- Store model lineage.
- Monitor drift.
- Keep rollback model ready.
- Protect sensitive training data.
- Remove low-quality or contradictory examples.
- Use RAG for changing knowledge.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Fine-tuning to add facts | Use RAG for knowledge |
| Beginner | Training on tiny low-quality data | Curate strong examples |
| Intermediate | No validation set | Hold out evaluation data |
| Intermediate | No baseline | Compare against prompting first |
| Production | No model versioning | Use model registry and rollback |
| Production | Training on sensitive data blindly | Apply privacy review and access controls |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is fine-tuning? | Additional training on task-specific examples to adapt model behavior. |
| Basic | What is LoRA? | A parameter-efficient method that trains small adapter weights instead of the full model. |
| Intermediate | What is QLoRA? | A memory-efficient fine-tuning method using quantization with LoRA. |
| Advanced | Fine-tuning vs RAG? | Use RAG for changing knowledge; use fine-tuning for behavior, style, format, or repeated task patterns. |
| Scenario | Model gives outdated policy. | Update the retrieval corpus and RAG pipeline, not the fine-tuned model. |

## 19. System Design Discussion

Fine-tuning belongs in a model lifecycle system:

- dataset creation
- quality review
- training
- evaluation
- model registry
- deployment
- monitoring
- rollback

Design decisions:

- fine-tune or prompt
- fine-tune or RAG
- full fine-tune or adapter
- hosted fine-tuning or local training
- dataset versioning
- deployment strategy

## 20. Hands-On Assignment

- Easy: List five tasks where fine-tuning is useful and five where RAG is better.
- Medium: Create 30 training examples for a support classifier.
- Hard: Design an evaluation set with edge cases and failure labels.

## 21. Mini Project

Build a Fine-Tuning Dataset Validator.

Requirements:

- Accept examples.
- Validate required fields.
- Detect empty outputs.
- Split train and validation sets.
- Produce a dataset quality report.

Folder structure:

```text
fine-tune-dataset-validator/
  app/
    main.py
    schemas.py
    validator.py
    splitter.py
  tests/
    test_validator.py
```

## 22. Production-Level Project

Build a Fine-Tuned Domain Assistant Lifecycle.

Real-world problem:

- A company wants consistent support responses with strict tone and format.

Architecture:

```text
[Examples] -> [Dataset Registry] -> [Fine-Tuning Job]
            -> [Evaluation Service] -> [Model Registry]
            -> [Model Gateway] -> [Production Monitoring]
```

Tech stack:

- FastAPI
- PostgreSQL
- object storage
- Redis/background workers
- model provider or local training stack
- evaluation dashboard

Scaling strategy:

- version every dataset and model
- automate evaluation
- deploy behind model gateway
- canary new model versions
- roll back on quality drop
- combine with RAG for factual grounding

## Quiz

1. What is fine-tuning?
2. What is LoRA?
3. What is QLoRA?
4. What is PEFT?
5. When is RAG better than fine-tuning?
6. Why is dataset quality important?
7. What is overfitting?
8. Why do you need a validation set?
9. What should a model registry store?
10. How would you deploy a fine-tuned model safely?

## Knowledge Check

You should be able to decide when fine-tuning is justified, explain LoRA/QLoRA/PEFT, and design a production lifecycle for fine-tuned models.

Are you ready for the next section?
