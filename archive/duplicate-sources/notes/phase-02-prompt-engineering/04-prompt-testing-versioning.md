# Prompt Testing and Versioning

## 1. Problem Statement

Prompt testing and versioning solve the problem of changing model behavior safely.

Prompts are not harmless text. In production, a prompt can affect customer support answers, legal extraction, financial summaries, tool calls, database writes, and user trust. If prompts are edited without testing and versioning, a small wording change can silently break an AI feature.

Real-world analogy: you would not deploy backend code without tests, version control, logs, and rollback. Production prompts deserve the same discipline.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Prompt testing checks prompt behavior against known cases. Prompt versioning tracks prompt changes over time. |
| Key terminology | prompt version, eval set, golden dataset, regression test, rubric, rollback, A/B test |
| Simple explanation | You test prompts before users depend on them, and you keep old versions so you can go back. |
| Mental model | Treat prompts like code and configuration. |
| Easy example | Test a support classifier prompt on 50 labeled tickets before deployment. |
| Use When | A prompt affects a product feature or automated workflow. |
| Avoid When | It is a temporary local experiment. |
| Advantages | Safer prompt changes, easier debugging, better quality tracking. |
| Tradeoffs | Requires test data and evaluation maintenance. |
| Limitations | Tests cannot cover every possible user input. |
| Production Example | A RAG assistant deploys a new answer prompt only after retrieval and answer-quality evals pass. |
| Interview Answer | Prompts should be tested and versioned because they are production artifacts that can regress like code. |

## 3. Intermediate Explanation

Prompt testing usually includes:

- representative inputs
- expected output or expected behavior
- negative cases
- edge cases
- evaluation criteria
- pass/fail thresholds

Types of tests:

| Test Type | Purpose | Example |
| --- | --- | --- |
| Format test | Checks structure | output must be valid JSON |
| Label test | Checks classification | refund ticket -> `refund_request` |
| Grounding test | Checks source usage | answer must cite source |
| Safety test | Checks refusal | prompt injection should not reveal hidden instructions |
| Regression test | Checks old bugs stay fixed | previous failure case still passes |
| Human review | Checks subjective quality | tone, helpfulness, correctness |

Data flow:

```text
Prompt version -> eval inputs -> model outputs -> evaluator -> score report -> deploy or reject
```

## 4. Advanced Explanation

Production prompt evaluation can use several layers:

1. Deterministic checks: JSON parse, required fields, allowed labels.
2. Heuristic checks: citation present, max length, banned phrase absent.
3. LLM-as-judge: useful but must be calibrated.
4. Human review: needed for high-risk or subjective quality.
5. Online metrics: user feedback, retry rate, escalation rate, conversion rate.

Optimization techniques:

- Start with 20-50 high-quality test cases.
- Add every production failure to the eval set.
- Track score by prompt version and model version.
- Separate dev, staging, and production prompts.
- Use canary rollout for risky changes.
- Keep rollback simple.

Performance considerations:

- Running evals costs tokens and time.
- LLM-as-judge can be inconsistent.
- Human review is slower but often higher quality.

Scaling considerations:

- Create a prompt registry.
- Run evals asynchronously.
- Store results for comparison.
- Add CI gates for critical prompts.
- Monitor quality after deployment.

Production challenges:

- user inputs are more diverse than eval sets
- models change behavior over time
- prompt improvements can help one segment and hurt another
- evaluation quality depends on test case quality

## 5. Internal Working

```text
Prompt change submitted
  |
  v
Prompt version created
  |
  v
Evaluation runner sends test cases to model
  |
  v
Outputs are validated and scored
  |
  v
Report compares old vs new version
  |
  v
Deploy, reject, or revise
  |
  v
Production monitoring watches real behavior
```

Detailed lifecycle:

1. Engineer edits prompt.
2. New prompt version is created.
3. Test suite runs.
4. Outputs are scored.
5. Reviewer checks failures.
6. Prompt is approved for staging.
7. Staging smoke tests run.
8. Prompt is released gradually.
9. Metrics are monitored.
10. Prompt is rolled back if quality drops.

## 6. When To Use

Use prompt testing and versioning for:

- customer-facing AI answers
- support classification
- RAG answer prompts
- structured extraction
- prompt injection defenses
- agent tool-use prompts
- legal or financial workflows
- anything that writes to a database or triggers actions

## 7. When NOT To Use

Do not overbuild prompt infrastructure for:

- a one-off learning notebook
- a throwaway demo
- early exploration before the task is known

Better alternatives at the beginning:

- a markdown table of test cases
- a small JSON file of examples
- manual comparison before building a full eval platform

## 8. Advantages

- Reduces production regressions.
- Makes prompt changes auditable.
- Helps compare prompt versions.
- Supports rollback.
- Improves interview and portfolio credibility.
- Creates a feedback loop from real failures to better tests.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Safety vs speed | Testing slows prompt changes but prevents regressions. |
| Coverage vs cost | More evals catch more issues but cost more. |
| Automation vs judgment | Automated scores help, but humans may be needed. |
| Flexibility vs governance | Versioning adds process but improves reliability. |

## 10. Limitations

- Eval sets are never complete.
- LLM judges may be biased or inconsistent.
- Human labels can disagree.
- Good evals require domain knowledge.
- Passing tests does not guarantee perfect production performance.

## 11. Real-World Examples

Startup example: a resume analyzer compares prompt versions on 100 resumes before release.

Enterprise example: a legal extraction prompt must pass schema validation and lawyer-reviewed examples.

FAANG-style example: a prompt platform tracks prompt version, model version, eval score, rollout, and user feedback at large scale.

Production system: a support bot adds every escalated bad answer to the regression suite.

## 12. Architecture Diagram

```text
[Prompt Author]
      |
      v
[Prompt Registry] -> [Eval Runner] -> [Score Report]
      |                                  |
      v                                  v
[Staging Prompt] ----------------> [Approval]
      |
      v
[Production Prompt] -> [Monitoring] -> [Rollback if needed]
```

## 13. Python Implementation

Test case model:

```python
from dataclasses import dataclass

@dataclass
class PromptTestCase:
    name: str
    input_text: str
    expected_label: str

test_cases = [
    PromptTestCase(
        name="refund request",
        input_text="I want to return my order and get my money back.",
        expected_label="refund_request",
    )
]
```

Simple evaluator:

```python
def score_label_prediction(predicted: str, expected: str) -> bool:
    return predicted.strip().lower() == expected.strip().lower()
```

Prompt version record:

```python
from dataclasses import dataclass

@dataclass
class PromptVersion:
    name: str
    version: str
    template: str
    model: str
    status: str
```

Regression report:

```python
def pass_rate(results: list[bool]) -> float:
    if not results:
        return 0.0
    return sum(results) / len(results)
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class PromptVersionCreate(BaseModel):
    name: str = Field(min_length=1)
    version: str = Field(min_length=1)
    template: str = Field(min_length=10)

class EvalSummary(BaseModel):
    prompt_name: str
    version: str
    total_cases: int
    passed_cases: int
    pass_rate: float

@app.post("/prompts/versions")
async def create_prompt_version(request: PromptVersionCreate) -> dict[str, str]:
    # In production, insert into PostgreSQL.
    return {"name": request.name, "version": request.version, "status": "draft"}

@app.post("/prompts/{name}/{version}/eval", response_model=EvalSummary)
async def run_prompt_eval(name: str, version: str) -> EvalSummary:
    # In production, load test cases, call model, score outputs, and store results.
    results = [True, True, False]
    return EvalSummary(
        prompt_name=name,
        version=version,
        total_cases=len(results),
        passed_cases=sum(results),
        pass_rate=pass_rate(results),
    )
```

Production-ready structure:

```text
app/
  api/routes/prompt_registry.py
  services/eval_runner.py
  services/prompt_registry.py
  models/prompt_models.py
  repositories/prompt_repository.py
  repositories/eval_repository.py
```

## 15. Database Integration

PostgreSQL tables:

```text
prompt_templates(id, name, current_version, owner, created_at)
prompt_versions(id, template_id, version, template_text, model, status, created_at)
prompt_test_cases(id, template_id, input_json, expected_json, tags)
prompt_eval_runs(id, prompt_version_id, model, total_cases, pass_rate, created_at)
prompt_eval_results(id, eval_run_id, test_case_id, output_json, passed, error)
```

Redis use:

- cache active production prompt
- throttle expensive eval runs
- store active rollout state

Monitoring store:

- prompt version
- model version
- latency
- cost
- validation failures
- user feedback

## 16. Production Considerations

- Never deploy critical prompt edits without evals.
- Log prompt version with every model call.
- Keep old versions available for rollback.
- Store model name and generation parameters with eval results.
- Add regression cases for every serious production bug.
- Use canary rollout for risky prompts.
- Separate offline evals from online monitoring.
- Protect test data if it contains sensitive examples.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Editing prompts directly inside route handlers | Use prompt templates |
| Beginner | Testing only one happy path | Include edge and failure cases |
| Intermediate | No prompt version in logs | Log prompt name and version |
| Intermediate | Comparing prompts by feeling only | Use test cases and metrics |
| Production | No rollback path | Keep previous versions deployable |
| Production | No online monitoring | Track real user feedback and validation failures |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | Why test prompts? | Prompt wording changes model behavior, so tests catch regressions before users see them. |
| Intermediate | What is a golden dataset? | A curated set of representative inputs and expected outputs used to evaluate prompts. |
| Intermediate | What should be logged for a prompt run? | Prompt version, model, parameters, input reference, output, latency, cost, validation result, and errors. |
| Advanced | How do you deploy prompt changes safely? | Version the prompt, run offline evals, test in staging, canary release, monitor metrics, and support rollback. |
| Scenario | A new prompt improves summaries but breaks JSON extraction. | Separate evals by task, rollback the extraction prompt, add failing cases, and avoid sharing one prompt across incompatible tasks. |

## 19. System Design Discussion

Prompt testing and versioning are part of AI CI/CD.

Design decisions:

- Store prompts in Git for engineering control or database for runtime updates.
- Use automated evals for format and labels.
- Use human review for subjective or high-risk outputs.
- Define pass thresholds before deployment.
- Link prompt version to model version.

A strong production system treats prompts as versioned configuration with tests, ownership, and rollback.

## 20. Hands-On Assignment

- Easy: Create 10 test cases for a support ticket classifier.
- Medium: Compare two prompt versions and calculate pass rate.
- Hard: Build an eval runner that stores results and blocks deployment below 90 percent pass rate.

## 21. Mini Project

Build a Prompt Evaluation CLI or API.

Requirements:

- Load prompt version.
- Load test cases.
- Run prompt against each case.
- Score outputs.
- Print pass rate.
- Save failed examples.

Folder structure:

```text
prompt-eval/
  app/
    prompts.py
    test_cases.py
    evaluator.py
    main.py
  tests/
    test_evaluator.py
```

## 22. Production-Level Project

Build a Prompt Registry and Evaluation Platform.

Real-world problem:

- Product teams need to change prompts safely without silently breaking AI features.

Architecture:

```text
[Prompt Editor] -> [Prompt Registry API] -> [PostgreSQL]
                         |
                         v
                   [Eval Runner Queue]
                         |
                         v
                   [Eval Results + Reports]
                         |
                         v
                   [Deployment Approval]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- Background worker
- pytest for deterministic tests
- optional LLM judge for qualitative scoring

Scaling strategy:

- Run evals asynchronously.
- Cache active prompt versions.
- Keep old prompt versions for rollback.
- Add prompt-level dashboards.
- Track cost by eval run and production feature.

## Quiz

1. Why should prompts be tested?
2. Why should prompts be versioned?
3. What is a golden dataset?
4. What is a regression test?
5. What is a prompt eval run?
6. Why log model version with prompt version?
7. What is canary rollout?
8. What is rollback?
9. What are the limits of LLM-as-judge?
10. How would you build prompt CI/CD?

## Knowledge Check

You should be able to explain why prompts are production artifacts and design a basic prompt testing, versioning, and rollback system.

Are you ready for the next section?
