# Temperature and Top-P

## 1. Problem Statement

LLMs generate text by choosing likely next tokens. Temperature and Top-P control how deterministic or creative that choice is.

Without these controls, every generation use case would have the same randomness level, which is not ideal.

Analogy: temperature is how adventurous the model is while choosing words; Top-P limits the menu of choices it can pick from.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Temperature and Top-P are generation parameters that control randomness. |
| Key terminology | sampling, probability, deterministic, creative, nucleus sampling |
| Simple explanation | Lower values make output more predictable; higher values make output more varied. |
| Mental model | Temperature changes confidence sharpness; Top-P limits choices to a probability group. |
| Easy example | Use low temperature for JSON extraction and higher temperature for brainstorming. |
| Use When | You need to tune consistency or creativity. |
| Avoid When | Do not use randomness settings as a substitute for factual grounding. |
| Advantages | Better control over style and variability. |
| Tradeoffs | More creativity can mean less consistency. |
| Limitations | Low temperature does not guarantee truth. |
| Production Example | Contract risk extraction uses low temperature for stable structured output. |
| Interview Answer | Temperature and Top-P affect sampling randomness, not the model's factual knowledge. |

## 3. Intermediate Explanation

Temperature adjusts the probability distribution:

- low temperature: sharper, more deterministic
- high temperature: flatter, more varied

Top-P, also called nucleus sampling, selects from the smallest set of tokens whose cumulative probability reaches `p`.

Common settings:

| Use Case | Temperature | Top-P |
| --- | --- | --- |
| JSON extraction | 0.0-0.3 | 0.8-1.0 |
| Summarization | 0.2-0.5 | 0.8-1.0 |
| Brainstorming | 0.7-1.0 | 0.9-1.0 |
| Coding | 0.1-0.4 | 0.8-1.0 |

## 4. Advanced Explanation

Production systems should test parameters using evaluation sets. Do not guess blindly.

Optimization techniques:

- Use low randomness for extraction.
- Use higher randomness for ideation.
- Keep parameters stable in production.
- Version prompt and parameter changes together.
- Evaluate outputs after parameter changes.

Performance considerations:

- These settings usually affect quality more than latency.
- Bad settings can increase retries and downstream validation failures.

## 5. Internal Working

```text
Model predicts token probabilities
  |
  v
Temperature reshapes probabilities
  |
  v
Top-P filters candidate tokens
  |
  v
Sampler selects next token
  |
  v
Repeat until stop condition
```

## 6. When To Use

Use low randomness for:

- extraction
- classification
- structured JSON
- factual answers
- deterministic workflows

Use higher randomness for:

- brainstorming
- creative writing
- alternative suggestions
- ideation

## 7. When NOT To Use

Do not increase temperature to make the model "smarter." Do not lower temperature and assume hallucinations disappear.

Better alternatives for factuality:

- RAG
- citations
- verification
- structured outputs
- validation

## 8. Advantages

- Gives control over consistency.
- Supports creative and deterministic use cases.
- Makes model behavior tunable.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Creativity vs consistency | Higher randomness gives variety but less repeatability. |
| Low randomness vs brittleness | Very low settings can still fail if prompt is bad. |
| Tuning vs maintenance | Parameter changes need testing and versioning. |

## 10. Limitations

- Does not guarantee truth.
- Does not replace prompt quality.
- Does not fix missing context.
- Provider implementations may differ.

## 11. Real-World Examples

Startup example: resume headline generator uses higher temperature for variations.

Enterprise example: invoice extraction uses low temperature for consistent JSON.

FAANG-style example: model gateway stores default generation settings per feature and evaluates changes before rollout.

Production system: support chatbot uses low temperature with RAG to reduce unexpected responses.

## 12. Architecture Diagram

```text
[Prompt] -> [LLM Token Probabilities] -> [Temperature] -> [Top-P] -> [Generated Token]
```

## 13. Python Implementation

Conceptual sampler:

```python
import random

def choose_token(candidates: list[tuple[str, float]]) -> str:
    tokens = [token for token, _ in candidates]
    weights = [weight for _, weight in candidates]
    return random.choices(tokens, weights=weights, k=1)[0]

print(choose_token([("yes", 0.8), ("maybe", 0.15), ("no", 0.05)]))
```

Parameter config:

```python
from dataclasses import dataclass

@dataclass
class GenerationConfig:
    temperature: float
    top_p: float
    max_output_tokens: int

json_extraction = GenerationConfig(temperature=0.1, top_p=0.9, max_output_tokens=800)
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class GenerationRequest(BaseModel):
    prompt: str
    temperature: float = Field(default=0.2, ge=0.0, le=2.0)
    top_p: float = Field(default=1.0, gt=0.0, le=1.0)

@app.post("/generation/config")
async def generation_config(request: GenerationRequest) -> dict[str, float | str]:
    return {
        "prompt": request.prompt,
        "temperature": request.temperature,
        "top_p": request.top_p,
    }
```

## 15. Database Integration

Store prompt version, model, temperature, Top-P, output, validation status, and user rating for evaluation.

## 16. Production Considerations

- Version model settings.
- Keep default settings per task type.
- Validate structured output.
- Run regression tests before changing parameters.
- Log parameter values with model calls.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Thinking temperature controls correctness | It controls randomness |
| Intermediate | Changing many parameters at once | Change one thing and evaluate |
| Production | No prompt/config versioning | Store prompt and generation config versions |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is temperature? | A parameter that controls randomness in token sampling. |
| Intermediate | What is Top-P? | It limits sampling to tokens within a cumulative probability mass. |
| Advanced | How tune parameters in production? | Use task-specific defaults, evaluation datasets, versioning, and monitoring. |
| Scenario | JSON output is inconsistent. | Lower randomness, improve schema instructions, use structured output mode, and validate. |

## 19. System Design Discussion

Generation parameters belong in configuration, not scattered throughout code. Production systems should know which feature uses which model and settings.

## 20. Hands-On Assignment

- Easy: Create configs for extraction and brainstorming.
- Medium: Build an API that validates temperature and Top-P.
- Hard: Compare outputs from three settings and score consistency.

## 21. Mini Project

Build a Model Parameter Playground that lets you compare low, medium, and high randomness outputs.

## 22. Production-Level Project

Build a prompt evaluation dashboard that records model settings, outputs, validation results, and human ratings.

## Quiz

1. What does temperature control?
2. What does Top-P control?
3. Which setting is better for JSON extraction?
4. Does low temperature guarantee factual accuracy?
5. What is sampling?
6. Why version model settings?
7. What is the tradeoff of high temperature?
8. How would you tune a summarizer?
9. How would you debug inconsistent output?
10. Why should parameter changes be evaluated?

## Knowledge Check

You should be able to choose reasonable generation settings for extraction, summarization, coding, and brainstorming.

Are you ready for the next section?