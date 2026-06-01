# Structured Outputs

## 1. Problem Statement

Structured outputs solve the problem of unreliable free-form model responses when an application needs predictable data.

Without structured outputs, developers often parse natural language with fragile string operations. That breaks easily when the model changes wording, adds extra explanation, omits fields, or returns invalid JSON.

Real-world analogy: if a backend service expects JSON, you do not want a human paragraph. You want a contract both sides can follow.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Structured output means asking or forcing the model to return data in a predictable schema such as JSON. |
| Key terminology | JSON, schema, Pydantic, parser, validation, enum, required field, retry |
| Simple explanation | Instead of "write anything," the model must return fields your code can read. |
| Mental model | Treat the LLM like an unreliable API and validate its response. |
| Easy example | Extract `{ "name": "Asha", "skills": ["Python", "FastAPI"] }` from a resume. |
| Use When | You need model output to feed another system. |
| Avoid When | The output is only final prose for a human. |
| Advantages | Easier parsing, testing, automation, and validation. |
| Tradeoffs | Less flexible and requires schema design. |
| Limitations | Valid JSON can still contain wrong facts. |
| Production Example | Contract analyzer extracts risk level, clause type, and evidence quote. |
| Interview Answer | Structured outputs make LLM responses machine-readable and safer for downstream automation, but still require validation. |

## 3. Intermediate Explanation

Structured output usually includes:

- a schema
- strict field names
- required and optional fields
- enums for fixed choices
- parser
- validator
- retry or fallback path

Common output types:

| Type | Example | Use Case |
| --- | --- | --- |
| Classification | `{ "intent": "refund" }` | routing |
| Extraction | `{ "invoice_id": "123" }` | document processing |
| Scoring | `{ "risk_score": 7 }` | review workflows |
| List of objects | `[{ "skill": "Python" }]` | resume parsing |
| Tool arguments | `{ "ticket_id": "T-1" }` | agents |

Data flow:

```text
Input text -> prompt + schema -> model output -> JSON parse -> Pydantic validation -> business validation
```

## 4. Advanced Explanation

Production structured output has two validation layers:

1. Schema validation: checks fields and types.
2. Business validation: checks whether values make sense.

Example:

- Schema valid: `{ "years_experience": 80 }`
- Business invalid: a normal candidate probably does not have 80 years of Python experience.

Optimization techniques:

- Use enums for limited labels.
- Use nullable fields instead of making the model invent unknown values.
- Include examples for tricky schemas.
- Keep schemas small.
- Split large extraction tasks into multiple passes.
- Retry only when the failure is recoverable.

Performance considerations:

- Complex schemas increase output length.
- Retries increase latency and cost.
- Large JSON outputs can exceed token budgets.

Production challenges:

- invalid JSON
- missing required fields
- hallucinated field values
- schema version migrations
- downstream systems trusting wrong values

## 5. Internal Working

```text
User/document input
  |
  v
Prompt builder inserts schema instructions
  |
  v
LLM returns JSON-like output
  |
  v
Parser attempts JSON decoding
  |
  v
Pydantic validates types and required fields
  |
  v
Business rules validate meaning
  |
  v
Accepted, retried, or sent for human review
```

Detailed lifecycle:

1. Define schema.
2. Build prompt with extraction rules.
3. Call model.
4. Parse JSON.
5. Validate with Pydantic.
6. Apply business checks.
7. Store raw input, raw output, parsed output, schema version, and validation status.
8. Retry or escalate when needed.

## 6. When To Use

Use structured outputs for:

- resume parsing
- invoice extraction
- contract risk analysis
- support ticket routing
- tool calling
- agent state updates
- compliance checklists
- database write suggestions

## 7. When NOT To Use

Avoid structured outputs when:

- the final answer is natural language only
- the structure is not used by code
- deterministic extraction can be done with regex or SQL
- the schema becomes too complex for one call

Better alternatives:

- normal parsers for known formats
- OCR plus deterministic field extraction
- rules engines
- human review workflows
- smaller multi-step extraction schemas

## 8. Advantages

- Easier to parse.
- Easier to test.
- Safer for workflows.
- Better integration with FastAPI and databases.
- Reduces fragile string parsing.
- Supports validation and retry logic.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Structure vs flexibility | Strict schemas reduce creative freedom. |
| Reliability vs latency | Validation and retries take time. |
| Schema detail vs token cost | Large schemas consume more prompt and output tokens. |
| Automation vs risk | Structured data can trigger actions, so validation matters. |

## 10. Limitations

- Valid schema does not mean true data.
- The model can omit uncertain values unless instructed.
- Complex nested schemas can fail more often.
- Schema changes need migration planning.
- Business validation still belongs in backend code.

## 11. Real-World Examples

Startup example: a resume analyzer extracts skills, role, seniority, and missing keywords.

Enterprise example: a contract analyzer extracts risky clauses and sends high-risk items for legal review.

FAANG-style example: internal AI workflows use typed intermediate states so agents can be tested and resumed.

Production system: an invoice processing service validates extracted fields before writing to accounting software.

## 12. Architecture Diagram

```text
[Raw Text]
    |
    v
[LLM Extractor Prompt + Schema]
    |
    v
[JSON Output]
    |
    v
[Parser] -> [Schema Validator] -> [Business Validator]
                                      |
                                      v
                              [Database / Workflow]
```

## 13. Python Implementation

Beginner Pydantic model:

```python
from pydantic import BaseModel

class ResumeSummary(BaseModel):
    candidate_name: str
    skills: list[str]
    years_experience: int
```

Validation example:

```python
data = {
    "candidate_name": "Asha",
    "skills": ["Python", "FastAPI"],
    "years_experience": 3,
}

summary = ResumeSummary.model_validate(data)
print(summary.skills)
```

Schema with enums:

```python
from typing import Literal
from pydantic import BaseModel, Field

class TicketClassification(BaseModel):
    intent: Literal["billing", "technical", "refund", "account_access"]
    confidence: float = Field(ge=0.0, le=1.0)
    reason: str
```

Business validation:

```python
def validate_resume_business_rules(summary: ResumeSummary) -> list[str]:
    errors: list[str] = []
    if summary.years_experience > 60:
        errors.append("years_experience is unrealistically high")
    if not summary.skills:
        errors.append("at least one skill is required")
    return errors
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class ResumeParseRequest(BaseModel):
    resume_text: str = Field(min_length=20)

class ResumeParseResponse(BaseModel):
    candidate_name: str
    skills: list[str]
    years_experience: int
    validation_warnings: list[str]

@app.post("/extract/resume", response_model=ResumeParseResponse)
async def extract_resume(request: ResumeParseRequest) -> ResumeParseResponse:
    # In production, call the LLM and parse its structured response.
    extracted = ResumeSummary(
        candidate_name="Unknown",
        skills=["Python"],
        years_experience=1,
    )
    warnings = validate_resume_business_rules(extracted)

    return ResumeParseResponse(
        candidate_name=extracted.candidate_name,
        skills=extracted.skills,
        years_experience=extracted.years_experience,
        validation_warnings=warnings,
    )
```

Production-ready structure:

```text
app/
  api/routes/extraction.py
  schemas/extraction.py
  services/llm_extractor.py
  services/output_validator.py
  repositories/extraction_repository.py
```

Error handling:

- `400`: invalid request
- `422`: model output failed validation
- `503`: model provider unavailable
- `202`: accepted for human review if uncertain

## 15. Database Integration

PostgreSQL fields:

```text
extraction_jobs(id, user_id, schema_name, schema_version, status, created_at)
extraction_results(id, job_id, raw_output, parsed_json, validation_errors, reviewed_at)
```

Store:

- raw input reference
- raw model output
- parsed structured output
- schema version
- validation status
- reviewer status

Redis use:

- cache schemas
- rate-limit extraction jobs
- store temporary retry counters

Vector database use:

- retrieve relevant policy or document context before structured extraction

## 16. Production Considerations

- Use schema versioning.
- Store raw model output for debugging.
- Never trust parsed data without validation.
- Add business validation after schema validation.
- Use retries only with limits.
- Send high-risk extractions for human review.
- Log validation failures by prompt version and model.
- Avoid storing sensitive raw text unless policy allows it.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Parsing free text with `split()` | Use JSON and schema validation |
| Beginner | Asking for "JSON" but not validating it | Always parse and validate |
| Intermediate | No enums for fixed labels | Use `Literal` or enum fields |
| Intermediate | One huge schema for everything | Split complex extraction into smaller schemas |
| Production | Treating valid JSON as true | Add business validation and human review |
| Production | Changing schema without migration | Version schemas and outputs |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is structured output? | A model response constrained to a predictable machine-readable format such as JSON. |
| Intermediate | Why use Pydantic? | It validates required fields, types, constraints, and makes output safer for app code. |
| Intermediate | What is the difference between schema and business validation? | Schema validation checks shape and types; business validation checks whether values make sense. |
| Advanced | How handle invalid model JSON in production? | Parse, validate, retry with limits, log failure, and fall back to human review or error response. |
| Scenario | The model returns valid JSON with an impossible value. | Accept schema validation but fail business validation, log it, and request correction or review. |

## 19. System Design Discussion

Structured outputs are critical when LLMs connect to backend systems. They convert the LLM from "text generator" into "probabilistic data extractor." Because it is probabilistic, backend validation remains mandatory.

Design decisions:

- Use structured output mode when provider supports it.
- Keep schemas simple.
- Store schema versions.
- Validate before writing to databases.
- Use human approval before high-impact automation.

## 20. Hands-On Assignment

- Easy: Create a Pydantic model for a support ticket classifier.
- Medium: Validate three JSON examples, including one invalid example.
- Hard: Add business validation and retry logic for failed extraction.

## 21. Mini Project

Build a Resume-to-JSON Extractor.

Requirements:

- Accept resume text.
- Return candidate name, skills, years, education, and suggested role.
- Validate all fields.
- Return validation warnings.
- Store raw and parsed outputs.

Folder structure:

```text
resume-extractor/
  app/
    main.py
    schemas.py
    extractor.py
    validator.py
  tests/
    test_resume_schema.py
```

## 22. Production-Level Project

Build a Contract Risk Extractor.

Real-world problem:

- Legal teams need structured risk extraction from contracts.

Architecture:

```text
[Contract Text] -> [Chunk/Retrieve Relevant Clauses] -> [LLM Extractor]
                                                     -> [Schema Validation]
                                                     -> [Business Validation]
                                                     -> [Human Review Queue]
```

Tech stack:

- FastAPI
- PostgreSQL
- Pydantic
- Redis for job state
- Vector DB if contracts are long

Scaling strategy:

- Process documents asynchronously.
- Store schema and prompt versions.
- Retry validation failures with limits.
- Escalate high-risk clauses.
- Monitor extraction accuracy by field.

## Quiz

1. What is structured output?
2. Why is free-form text hard for backend systems?
3. What does Pydantic validate?
4. What is business validation?
5. Can valid JSON still be wrong?
6. Why use enums for classification?
7. Why store raw model output?
8. What is schema versioning?
9. When should extraction go to human review?
10. How would you handle invalid JSON from a model?

## Knowledge Check

You should be able to design a structured output schema, validate model responses, and explain why schema validation alone is not enough for production.

Are you ready for the next section?
