# CI/CD for AI Systems

## 1. Problem Statement

CI/CD for AI systems solves the problem of safely testing, validating, building, and deploying code, prompts, retrieval settings, schemas, and model configuration.

Traditional CI/CD checks code. AI systems also need checks for prompts, structured outputs, retrieval quality, model versions, token cost, and safety behavior.

Without AI-aware CI/CD:

- prompt edits can silently break outputs
- retrieval changes can reduce answer quality
- schema changes can break clients
- model upgrades can regress behavior
- deployments may be hard to roll back

Real-world analogy: AI CI/CD is like a quality gate at a factory. It checks not only whether the machine turns on, but whether the product still meets quality standards.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | CI/CD automates checks, builds, and deployments; AI CI/CD adds prompt, retrieval, model, and safety evaluation. |
| Key terminology | CI, CD, pipeline, test, eval, golden dataset, smoke test, canary, rollback |
| Simple explanation | Test changes before release and deploy safely. |
| Mental model | Treat prompts, models, retrieval configs, and schemas like deployable artifacts. |
| Easy example | Run prompt regression tests before deploying a new prompt version. |
| Use When | Shipping production AI applications. |
| Avoid When | Doing a quick local experiment. |
| Advantages | Safer releases, fewer regressions, faster iteration. |
| Tradeoffs | Requires tests, datasets, infrastructure, and maintenance. |
| Limitations | Tests cannot catch every AI behavior. |
| Production Example | RAG deployment is blocked if retrieval eval recall drops below threshold. |
| Interview Answer | AI CI/CD should validate code plus prompts, schemas, retrieval quality, model settings, safety behavior, and deployment health. |

## 3. Intermediate Explanation

CI/CD stages:

| Stage | Purpose |
| --- | --- |
| lint/type checks | code quality |
| unit tests | function behavior |
| API tests | endpoint contracts |
| schema tests | request/response compatibility |
| prompt evals | output quality and format |
| retrieval evals | source recall and precision |
| safety tests | injection/refusal behavior |
| build image | create deployment artifact |
| staging deploy | test realistic environment |
| smoke test | verify critical flows |
| production rollout | deploy gradually |
| monitoring | detect regressions |

AI artifacts to version:

- prompt templates
- retrieval settings
- chunking strategy
- embedding model
- LLM model name
- generation parameters
- output schemas
- evaluation datasets

## 4. Advanced Explanation

AI systems need both deterministic tests and probabilistic evaluations.

Optimization techniques:

- keep a golden dataset for core tasks
- add regression cases from production failures
- run fast checks on every commit
- run expensive evals on release candidates
- use canary rollouts for risky model or prompt changes
- store eval scores by version
- block deployments below thresholds
- keep rollback path for prompt/config/model versions

Performance considerations:

- LLM evals cost tokens and time
- retrieval evals need indexed test data
- canary monitoring takes longer than unit tests
- flaky LLM judge tests need thresholds and review

Production challenges:

- non-deterministic model outputs
- hidden provider model changes
- stale evaluation datasets
- testing private data safely
- prompt and model version mismatch
- rollback across code and config

## 5. Internal Working

```text
Developer changes code/prompt/config
  |
  v
CI runs code tests and AI evals
  |
  v
Docker image is built
  |
  v
Staging deployment runs smoke tests
  |
  v
Production canary rollout starts
  |
  v
Monitoring checks metrics
  |
  v
Promote or rollback
```

Detailed lifecycle:

1. Pull request is opened.
2. Unit and API tests run.
3. Prompt and schema tests run.
4. Retrieval evals run if retrieval changed.
5. Docker image is built.
6. Staging deploy happens.
7. Smoke tests verify key flows.
8. Production rollout starts gradually.
9. Metrics are monitored.
10. Rollback happens if errors or quality regressions appear.

## 6. When To Use

Use AI CI/CD when:

- prompts affect users
- RAG retrieval affects answers
- model versions can change
- structured outputs feed workflows
- deployments serve real users
- compliance or reliability matters

## 7. When NOT To Use

Avoid heavy CI/CD when:

- building a throwaway notebook
- validating an idea locally
- no users depend on the system yet

Better early alternatives:

- local tests
- markdown eval table
- manual smoke checklist
- simple deployment script

## 8. Advantages

- Reduces regressions.
- Makes releases repeatable.
- Enables rollback.
- Catches prompt/schema failures.
- Tracks AI quality over time.
- Improves interview credibility.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Safety vs speed | More gates slow releases but reduce failures. |
| Eval coverage vs cost | More AI evals cost time and money. |
| Automation vs human judgment | Some quality checks still need review. |
| Strict thresholds vs flexibility | Thresholds can block useful changes if poorly designed. |

## 10. Limitations

- Tests cannot cover all user inputs.
- AI outputs may vary.
- LLM-as-judge can be inconsistent.
- Eval datasets can become stale.
- Passing CI does not guarantee production quality.

## 11. Real-World Examples

Startup example: CI runs API tests and prompt JSON validation before deploying.

Enterprise example: RAG changes require retrieval recall checks over a curated document set.

FAANG-style example: release pipelines include offline evals, canaries, live metrics, automatic rollback, and human review gates.

Production system: prompt version, model version, chunking version, eval score, and deployment ID are logged for every release.

## 12. Architecture Diagram

```text
[Pull Request]
    |
    v
[CI Pipeline]
    +-> code tests
    +-> prompt evals
    +-> retrieval evals
    +-> safety tests
    |
    v
[Build Docker Image]
    |
    v
[Deploy Staging]
    |
    v
[Smoke Tests]
    |
    v
[Canary Production]
    |
    v
[Monitor + Rollback]
```

## 13. Python Implementation

Prompt contract test:

```python
def test_prompt_requires_json() -> None:
    prompt = "Return valid JSON with fields: name, score, reason."
    assert "JSON" in prompt
    assert "name" in prompt
```

Schema test:

```python
from pydantic import BaseModel

class ExtractedRisk(BaseModel):
    risk_level: str
    reason: str

def test_schema_accepts_valid_output() -> None:
    output = {"risk_level": "high", "reason": "Missing termination clause"}
    parsed = ExtractedRisk.model_validate(output)
    assert parsed.risk_level == "high"
```

Eval score gate:

```python
def deployment_allowed(pass_rate: float, min_pass_rate: float = 0.9) -> bool:
    return pass_rate >= min_pass_rate
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class EvalGateRequest(BaseModel):
    eval_name: str
    pass_rate: float = Field(ge=0, le=1)
    min_required: float = Field(default=0.9, ge=0, le=1)

@app.post("/deploy/eval-gate")
async def eval_gate(request: EvalGateRequest) -> dict[str, str]:
    if not deployment_allowed(request.pass_rate, request.min_required):
        raise HTTPException(status_code=422, detail="eval gate failed")
    return {"status": "approved"}
```

Production-ready structure:

```text
.github/workflows/
  ci.yml
deployment/
  smoke_tests.py
app/
  services/evaluation_service.py
  services/deployment_gate.py
tests/
  test_api_contracts.py
  test_prompt_contracts.py
evals/
  retrieval_eval_cases.json
  prompt_eval_cases.json
```

## 15. Database Integration

PostgreSQL:

```text
deployments(id, version, image_tag, status, deployed_at)
eval_runs(id, deployment_id, eval_name, pass_rate, report_uri, created_at)
prompt_versions(id, name, version, status)
model_configs(id, model_name, parameters_json, status)
```

Object storage:

- eval reports
- logs
- test datasets

Redis:

- deployment lock
- canary rollout state

## 16. Production Considerations

- Version prompts and model configs.
- Store eval reports.
- Add rollback for code and prompt config.
- Run smoke tests after deploy.
- Use canaries for risky changes.
- Add security tests for prompt injection.
- Test database migrations.
- Keep staging close to production.
- Monitor live metrics after release.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | No tests | Add API and service tests |
| Beginner | Only testing code | Test prompts and schemas too |
| Intermediate | No eval dataset | Maintain golden examples |
| Intermediate | No smoke tests | Verify critical flows after deploy |
| Production | No rollback | Keep previous versions deployable |
| Production | No canary | Roll out risky AI changes gradually |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is CI/CD? | Automated testing, building, and deployment of software changes. |
| Basic | What is different for AI CI/CD? | Prompts, retrieval configs, model versions, schemas, and safety behavior also need evaluation. |
| Intermediate | What is a golden dataset? | A curated set of representative examples used to test AI behavior. |
| Advanced | How deploy AI changes safely? | Run code tests, prompt evals, retrieval evals, staging smoke tests, canary rollout, monitoring, and rollback. |
| Scenario | New prompt breaks JSON output. | Roll back prompt version, add failing example to eval set, and improve schema validation. |

## 19. System Design Discussion

AI CI/CD should treat these as deployable assets:

- code
- prompts
- schemas
- retrieval configs
- model configs
- chunking versions
- embedding versions

Design decisions:

- which evals run per commit
- which evals run per release
- thresholds
- rollback mechanism
- canary strategy
- human approval gates

## 20. Hands-On Assignment

- Easy: Write a prompt contract test.
- Medium: Create a smoke test for a chat endpoint.
- Hard: Design a CI pipeline that blocks deployment if retrieval recall drops.

## 21. Mini Project

Build an AI CI Checklist.

Requirements:

- Code tests.
- Prompt tests.
- Schema tests.
- Retrieval evals.
- Safety tests.
- Smoke tests.
- Rollback checklist.

Folder structure:

```text
ai-ci-checklist/
  README.md
  tests/
  evals/
  deployment/
```

## 22. Production-Level Project

Build CI/CD for a RAG Platform.

Real-world problem:

- RAG platform changes must deploy without breaking retrieval, prompts, or APIs.

Architecture:

```text
[PR] -> [CI Tests + AI Evals] -> [Docker Build]
     -> [Staging Deploy] -> [Smoke Tests]
     -> [Canary] -> [Production Monitoring]
```

Tech stack:

- GitHub Actions or similar
- Docker
- FastAPI
- pytest
- eval datasets
- deployment platform

Scaling strategy:

- separate fast and slow evals
- cache dependencies
- store eval history
- run canaries for high-risk changes
- monitor production regressions

## Quiz

1. What is CI?
2. What is CD?
3. Why do prompts need tests?
4. What is a golden dataset?
5. What is a smoke test?
6. What is a canary release?
7. Why version model configs?
8. What is rollback?
9. Why can AI tests be flaky?
10. How would you design CI/CD for RAG?

## Knowledge Check

You should be able to design an AI-aware CI/CD pipeline that tests code, prompts, schemas, retrieval, safety, deployments, and rollback.

Are you ready for the next section?
