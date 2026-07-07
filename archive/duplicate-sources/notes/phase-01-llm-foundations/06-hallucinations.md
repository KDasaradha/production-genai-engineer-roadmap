# Hallucinations

## 1. Problem Statement

LLMs can produce confident text that is false, unsupported, outdated, or invented. This is called hallucination.

Without understanding hallucinations, teams may trust AI output blindly and ship unsafe systems.

Analogy: a very fluent intern writes a polished answer even when they are guessing.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | A hallucination is an AI-generated statement that is incorrect, unsupported, or fabricated. |
| Key terminology | factuality, grounding, citations, retrieval, verification, uncertainty |
| Simple explanation | The model can sound confident even when it is wrong. |
| Mental model | LLMs predict likely text, not guaranteed truth. |
| Easy example | The model invents a library function that does not exist. |
| Use When | Always consider hallucination risk in user-facing AI systems. |
| Avoid When | Never use raw LLM output for high-stakes decisions without validation. |
| Advantages | Knowing the risk helps you design safer systems. |
| Tradeoffs | More safety checks add latency and complexity. |
| Limitations | Hallucinations cannot be eliminated completely. |
| Production Example | RAG answer must cite retrieved source chunks. |
| Interview Answer | Hallucinations happen because LLMs generate probable text and may lack grounding, current facts, or constraints. |

## 3. Intermediate Explanation

Common causes:

- missing context
- outdated training data
- ambiguous prompt
- pressure to answer
- poor retrieval
- conflicting context
- no verification step

Types:

| Type | Example |
| --- | --- |
| Factual | Invented policy or date |
| Citation | Fake source or wrong page |
| Code | Nonexistent function or package |
| Reasoning | Confident but invalid logic |
| Retrieval-grounded | Answer ignores retrieved context |

## 4. Advanced Explanation

Mitigation techniques:

- RAG with citations.
- Refusal when context is insufficient.
- Structured outputs and validation.
- Tool calls for live data.
- Deterministic workflows for critical tasks.
- Human review for high-risk use cases.
- Evaluation datasets.
- Retrieval quality monitoring.

Production challenges:

- RAG can still hallucinate.
- Citations can be wrong.
- Users may overtrust fluent answers.
- Safety requirements differ by domain.

## 5. Internal Working

```text
Prompt
  |
  v
Model predicts likely continuation
  |
  v
If context is missing or weak, model may still continue
  |
  v
Output sounds fluent
  |
  v
System must verify, ground, or constrain output
```

## 6. When To Use

Hallucination mitigation is required for:

- legal analysis
- medical workflows
- finance
- customer support
- enterprise knowledge assistants
- code generation
- compliance

## 7. When NOT To Use

Do not overbuild heavy mitigation for low-risk creative brainstorming where factual correctness is not the goal.

Better alternatives:

- Human review.
- Search tools.
- Rules engines.
- Database lookups.
- Deterministic validation.

## 8. Advantages

Understanding hallucinations improves:

- product safety
- user trust
- interview readiness
- production architecture
- evaluation design

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Safety vs latency | Verification adds steps. |
| Grounding vs creativity | Strict grounding can reduce open-ended generation. |
| Cost vs reliability | More retrieval and validation costs more. |
| UX vs honesty | Refusals are safer but may frustrate users. |

## 10. Limitations

- No mitigation removes hallucination risk completely.
- Evaluation sets may miss rare failures.
- Grounding depends on source quality.
- Human reviewers can also make mistakes.

## 11. Real-World Examples

Startup example: AI support bot must not invent refund policy.

Enterprise example: internal assistant must cite source documents and say when it cannot find an answer.

FAANG-style example: assistant answer quality pipeline logs unsupported claims and uses human evaluation.

Production system: RAG response generator is instructed to answer only from retrieved context and return "not enough information" otherwise.

## 12. Architecture Diagram

```text
[User Question]
      |
      v
[Retriever] -> [Relevant Sources]
      |
      v
[LLM with Grounding Instructions]
      |
      v
[Citation/Validation Check]
      |
      v
[Answer or Refusal]
```

## 13. Python Implementation

Basic guard:

```python
def require_context_answer(question: str, context: str) -> str:
    if not context.strip():
        return "I do not have enough information to answer from the provided context."
    return f"Answer the question using only this context:\n{context}\n\nQuestion: {question}"
```

Simple citation check:

```python
def has_citation(answer: str) -> bool:
    return "[source:" in answer.lower()
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class GroundedAnswerRequest(BaseModel):
    question: str
    context: str

@app.post("/answer/grounded")
async def grounded_answer(request: GroundedAnswerRequest) -> dict[str, str]:
    if not request.context.strip():
        raise HTTPException(status_code=422, detail="context is required for grounded answers")
    prompt = require_context_answer(request.question, request.context)
    return {"prompt": prompt}
```

## 15. Database Integration

Store source documents, retrieved chunks, generated answers, citations, validation results, and user feedback.

## 16. Production Considerations

- Log retrieved context and answer references.
- Add refusal behavior.
- Validate citations.
- Use monitoring for low-confidence or no-source answers.
- Use human review for high-risk domains.
- Track hallucination reports.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Trusting confident wording | Verify important claims |
| Intermediate | Thinking RAG eliminates hallucinations | Treat RAG as risk reduction, not a guarantee |
| Production | No feedback loop | Collect user reports and evaluate failures |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is hallucination? | A false or unsupported model-generated statement. |
| Intermediate | Why does it happen? | LLMs generate likely text and may lack grounding or current facts. |
| Advanced | How reduce hallucinations? | Use RAG, citations, validation, refusal rules, tools, and evaluation. |
| Scenario | RAG bot invents policy. | Check retrieval, context quality, prompt constraints, citation validation, and logs. |

## 19. System Design Discussion

Hallucination mitigation belongs across the architecture: retrieval quality, prompt constraints, model settings, validation, UX, monitoring, and human escalation.

## 20. Hands-On Assignment

- Easy: Collect five hallucination examples.
- Medium: Write a grounded-answer prompt.
- Hard: Build a citation validator for generated answers.

## 21. Mini Project

Build a Hallucination Case Study Notebook with examples, causes, and mitigations.

## 22. Production-Level Project

Add hallucination reporting and review workflow to a RAG support bot.

## Quiz

1. What is a hallucination?
2. Why can LLMs sound confident when wrong?
3. Does RAG eliminate hallucinations?
4. What is grounding?
5. What are citations useful for?
6. What is refusal behavior?
7. How can poor retrieval cause hallucination?
8. Why validate structured outputs?
9. What logs help debug hallucinations?
10. When is human review necessary?

## Knowledge Check

You should be able to explain hallucination risk and design at least three mitigation layers.

Are you ready for the next section?