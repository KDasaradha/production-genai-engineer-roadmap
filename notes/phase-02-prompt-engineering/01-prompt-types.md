# Prompt Types

## 1. Problem Statement

Prompt types solve the problem of telling an LLM what to do with the right amount of instruction and examples.

If prompt types do not exist, every request becomes a vague instruction like "do this task." That leads to inconsistent format, wrong assumptions, poor reasoning boundaries, and outputs that are hard to use in real applications.

Real-world analogy: if you hire a new developer, sometimes a short task description is enough. Sometimes you need one example. Sometimes you need several examples showing edge cases. Prompting works the same way.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Prompt types are patterns for instructing an LLM, commonly zero-shot, one-shot, and few-shot prompting. |
| Key terminology | instruction, context, example, zero-shot, one-shot, few-shot, output format |
| Simple explanation | A prompt can ask directly, show one example, or show multiple examples before asking the model to solve a new case. |
| Mental model | You are teaching the model the task for this request using words and examples. |
| Easy example | "Classify this text as positive, neutral, or negative" is zero-shot. Adding examples makes it one-shot or few-shot. |
| Use When | You need quick task control without training a model. |
| Avoid When | Exact deterministic business logic is better handled by code. |
| Advantages | Fast, flexible, cheap to change, and useful before fine-tuning. |
| Tradeoffs | Uses tokens and may behave differently across models. |
| Limitations | Examples guide behavior but do not guarantee correctness. |
| Production Example | A support system uses few-shot examples to classify ticket intent. |
| Interview Answer | Prompt types control model behavior by varying how much instruction and example context the model receives. |

## 3. Intermediate Explanation

The three core prompt types are:

| Type | Meaning | Best For | Risk |
| --- | --- | --- | --- |
| Zero-shot | No examples, only instruction | Simple tasks, broad models | Ambiguous output |
| One-shot | One example | Simple format imitation | One example may bias too much |
| Few-shot | Multiple examples | Classification, extraction, style, edge cases | More token cost |

Prompt components:

- Task: what the model should do.
- Context: information the model should use.
- Constraints: rules the model must follow.
- Examples: input-output demonstrations.
- Output format: expected response shape.

Data flow:

```text
Instruction -> examples if needed -> user input -> model output -> validation
```

Practical examples:

- Classify support tickets.
- Extract fields from resumes.
- Rewrite emails in a polite tone.
- Summarize meeting notes.
- Generate SQL from natural language after examples.

Industry usage:

- Customer support automation.
- Legal and contract review.
- HR resume screening.
- Content moderation support.
- AI coding assistants.

## 4. Advanced Explanation

Prompt type selection should depend on task complexity, failure cost, model capability, and output strictness.

Optimization techniques:

- Start zero-shot for simple tasks.
- Add examples only when the model misunderstands format or boundaries.
- Include edge-case examples, not only obvious examples.
- Keep examples short and representative.
- Separate instructions from examples.
- Version examples as part of the prompt.

Performance considerations:

- Few-shot prompts increase token cost and latency.
- Too many examples can distract the model.
- Poor examples can teach the wrong behavior.
- Examples from one model may not transfer perfectly to another.

Scaling considerations:

- Store prompts in a registry instead of hardcoding them everywhere.
- Track prompt version with every model call.
- Use evaluation sets before changing examples.
- Use structured outputs when the application needs typed data.

Production challenges:

- Prompt drift after edits.
- User inputs that conflict with examples.
- Edge cases not covered by examples.
- Prompt injection inside user-provided text.

## 5. Internal Working

```text
Input
  |
  v
Prompt builder combines:
  - task instruction
  - constraints
  - examples
  - user input
  |
  v
LLM predicts response based on full prompt
  |
  v
Application validates format and meaning
  |
  v
Output returned or retried
```

Detailed lifecycle:

1. Product defines the task.
2. Engineer writes a zero-shot prompt.
3. Test examples reveal weaknesses.
4. Engineer adds one-shot or few-shot examples.
5. Output is validated.
6. Prompt is versioned.
7. Production logs track prompt version, model, parameters, and result.

## 6. When To Use

Use prompt types when:

- You are starting a new LLM feature.
- You need to shape output format.
- The task is language-heavy.
- You need quick iteration.
- You want to avoid fine-tuning initially.

Ideal use cases:

- Sentiment classification.
- Resume analysis.
- Contract clause extraction.
- Email rewriting.
- Summarization.
- Ticket routing.

## 7. When NOT To Use

Avoid prompt-only solutions when:

- Exact rules are required.
- The task is simple enough for code.
- The output triggers risky actions.
- The model needs private/current knowledge without retrieval.
- The expected output must be guaranteed valid.

Better alternatives:

- Normal Python logic for deterministic rules.
- RAG for external knowledge.
- Structured output mode for JSON.
- Fine-tuning for repeated behavior after prompt limits are proven.

## 8. Advantages

- Very fast to prototype.
- No training dataset required.
- Works with hosted LLM APIs.
- Easy to compare prompt variants.
- Useful for both beginner demos and production systems.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Simplicity vs reliability | Simple prompts are easy but may be inconsistent. |
| Examples vs token cost | More examples guide behavior but increase cost. |
| Flexibility vs control | Prompts are flexible but less strict than code. |
| Speed vs evaluation | Fast prompt edits can break production without tests. |

## 10. Limitations

- Prompts cannot guarantee truth.
- Examples cannot cover every input.
- Prompt behavior can change after model upgrades.
- Long prompts can exceed context limits.
- Prompt-only systems are vulnerable to injection if user input is not handled carefully.

## 11. Real-World Examples

Startup example: a resume analyzer uses few-shot examples to classify candidate seniority.

Enterprise example: a support system classifies tickets as billing, technical, refund, or account access.

FAANG-style example: a model gateway stores task-specific prompt templates with versions, evaluation scores, and rollout status.

Production system: a contract analyzer uses few-shot examples plus schema validation to extract clauses and risk levels.

## 12. Architecture Diagram

```text
[User Input]
      |
      v
[Prompt Template Store] -> [Prompt Builder]
                              |
                              v
                            [LLM]
                              |
                              v
                      [Validator / Parser]
                              |
                              v
                         [App Response]
```

## 13. Python Implementation

Beginner zero-shot:

```python
def zero_shot_prompt(text: str) -> str:
    return f"Classify the sentiment as positive, neutral, or negative.\n\nText: {text}\nSentiment:"
```

One-shot:

```python
def one_shot_prompt(text: str) -> str:
    return f"""Classify the sentiment as positive, neutral, or negative.

Example:
Text: I love this product.
Sentiment: positive

Text: {text}
Sentiment:"""
```

Few-shot:

```python
def few_shot_prompt(text: str) -> str:
    return f"""Classify the support ticket intent.

Examples:
Text: I was charged twice this month.
Intent: billing_issue

Text: I cannot reset my password.
Intent: account_access

Text: I want to return my order.
Intent: refund_request

Text: {text}
Intent:"""
```

Production-style prompt object:

```python
from dataclasses import dataclass

@dataclass
class PromptTemplate:
    name: str
    version: str
    template: str

    def render(self, **values: str) -> str:
        return self.template.format(**values)

ticket_prompt = PromptTemplate(
    name="ticket-intent-classifier",
    version="v1",
    template="Classify this ticket into billing, technical, refund, or account_access.\nTicket: {ticket}\nIntent:",
)

print(ticket_prompt.render(ticket="I need help with my invoice."))
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class ClassificationRequest(BaseModel):
    text: str = Field(min_length=1)
    mode: str = "few-shot"

class ClassificationPromptResponse(BaseModel):
    prompt: str
    prompt_type: str
    prompt_version: str

@app.post("/prompts/classification", response_model=ClassificationPromptResponse)
async def build_classification_prompt(request: ClassificationRequest) -> ClassificationPromptResponse:
    if request.mode not in {"zero-shot", "one-shot", "few-shot"}:
        raise HTTPException(status_code=400, detail="unsupported prompt mode")

    if request.mode == "zero-shot":
        prompt = zero_shot_prompt(request.text)
    elif request.mode == "one-shot":
        prompt = one_shot_prompt(request.text)
    else:
        prompt = few_shot_prompt(request.text)

    return ClassificationPromptResponse(
        prompt=prompt,
        prompt_type=request.mode,
        prompt_version="v1",
    )
```

Production-ready structure:

```text
app/
  api/routes/prompts.py
  services/prompt_builder.py
  models/prompt_models.py
  repositories/prompt_repository.py
  tests/test_prompt_builder.py
```

## 15. Database Integration

Store:

- prompt name
- prompt version
- prompt type
- template text
- examples
- model name
- generation settings
- test score
- deployment status

PostgreSQL table idea:

```text
prompt_templates(id, name, version, type, template, status, created_at)
prompt_runs(id, prompt_id, model, input_hash, output, latency_ms, passed_validation)
```

Redis use:

- cache active prompt templates
- rate-limit prompt testing endpoints

## 16. Production Considerations

- Version every prompt.
- Log prompt name and version with each model call.
- Keep examples free of secrets and personal data.
- Validate output after model generation.
- Test prompts against edge cases.
- Use rollback when a prompt performs worse.
- Keep prompt construction separate from route handlers.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Writing vague prompts | Include task, constraints, and output format |
| Beginner | Thinking one prompt works for everything | Choose zero-shot, one-shot, or few-shot based on task |
| Intermediate | Adding too many examples | Use representative examples only |
| Intermediate | Mixing user input and instructions unclearly | Separate system/task instructions from user content |
| Production | No prompt versioning | Store and log prompt versions |
| Production | No evaluation dataset | Maintain test cases before deployment |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is zero-shot prompting? | Asking the model to perform a task without giving examples. |
| Basic | What is few-shot prompting? | Giving multiple input-output examples before asking the model to solve a new case. |
| Intermediate | When do examples help? | When the model needs format, tone, labels, or edge-case boundaries. |
| Advanced | How do you manage prompts in production? | Store prompts as versioned artifacts, test them with eval cases, log versions, monitor quality, and support rollback. |
| Scenario | Ticket classification is inconsistent. | Add representative examples, constrain labels, validate output, and test against a labeled dataset. |

## 19. System Design Discussion

Prompt templates are part of the application contract. In a serious AI backend, prompts should not be random strings hidden inside route handlers.

Design decisions:

- Store prompts in code for simplicity or database for dynamic updates.
- Use few-shot examples only when they improve measured quality.
- Validate output labels before writing to downstream systems.
- Keep prompt version linked to model version and generation settings.

## 20. Hands-On Assignment

- Easy: Write zero-shot, one-shot, and few-shot prompts for sentiment classification.
- Medium: Create a support ticket intent classifier prompt with 8 examples.
- Hard: Build a small test set and compare prompt accuracy manually.

## 21. Mini Project

Build a Prompt Playground.

Requirements:

- Accept user input.
- Let user choose zero-shot, one-shot, or few-shot mode.
- Build and display the final prompt.
- Store prompt runs locally in a markdown table or JSON file.
- Compare outputs manually.

Folder structure:

```text
prompt-playground/
  app/
    main.py
    prompt_builder.py
    schemas.py
  tests/
    test_prompt_builder.py
```

## 22. Production-Level Project

Build a Prompt Registry Service.

Real-world problem:

- Teams need to safely manage prompt templates used by production AI features.

Architecture:

```text
[Admin/User] -> [FastAPI Prompt Registry] -> [PostgreSQL]
                                      |
                                      v
                               [Evaluation Runner]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis cache
- Pydantic
- pytest

Scaling strategy:

- Cache active prompt versions.
- Keep old versions for rollback.
- Add async evaluation jobs.
- Log usage by feature and tenant.

## Quiz

1. What is zero-shot prompting?
2. What is one-shot prompting?
3. What is few-shot prompting?
4. Why do examples improve output format?
5. What is the downside of too many examples?
6. Why should prompts be versioned?
7. When is deterministic Python code better than prompting?
8. Why should prompt output be validated?
9. What should be stored for each prompt run?
10. How would you debug inconsistent prompt output?

## Knowledge Check

You should be able to choose a prompt type, explain the tradeoffs, build a prompt template, and describe how prompts should be managed in production.

Are you ready for the next section?
