# Context Windows

## 1. Problem Statement

An LLM can only process a limited number of tokens at once. The context window is that limit. It solves the boundary problem: how much input and output can fit in one model call.

Without context windows, systems would assume models can read unlimited documents and chat history, which is false.

Analogy: a context window is the model's desk space. If too many papers are on the desk, something must be removed.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | A context window is the maximum number of tokens a model can process in one request, including input and output. |
| Key terminology | context, prompt, completion, truncation, memory, token budget |
| Simple explanation | The model can only "see" what fits into the current request. |
| Mental model | Limited working memory. |
| Easy example | If chat history is too long, older messages may need summarization. |
| Use When | Designing prompts, chat memory, RAG, summarization, and long-document workflows. |
| Avoid When | Do not treat a large context window as a replacement for retrieval design. |
| Advantages | Makes model input bounded and manageable. |
| Tradeoffs | Bigger context can cost more and increase latency. |
| Limitations | More context does not guarantee better reasoning or answer quality. |
| Production Example | RAG sends only the top relevant chunks instead of the whole knowledge base. |
| Interview Answer | The context window is the token limit for what the model can consider in a single call. |

## 3. Intermediate Explanation

The context window includes:

- system instructions
- developer instructions
- user message
- chat history
- retrieved documents
- tool outputs
- expected model output

Data flow:

```text
Instructions + history + retrieved context + user query + reserved output <= context window
```

## 4. Advanced Explanation

Longer context can help, but it also creates risks:

- higher cost
- higher latency
- attention dilution
- accidental instruction conflicts
- larger attack surface for prompt injection
- harder debugging

Optimization techniques:

- summarize old chat history
- retrieve relevant chunks
- compress context
- remove boilerplate
- rank information by importance
- reserve output budget

## 5. Internal Working

```text
User query
  |
  v
Collect instructions, memory, retrieved docs
  |
  v
Count or estimate tokens
  |
  v
Trim, summarize, or reject if too large
  |
  v
Send bounded prompt to model
  |
  v
Receive output
```

## 6. When To Use

Think about context windows when:

- building chat memory
- doing document Q&A
- summarizing long text
- designing agents with tool outputs
- estimating cost
- preventing prompt overflow

## 7. When NOT To Use

Do not solve every long input problem by choosing the largest context model. Often better alternatives are:

- RAG
- summarization
- hierarchical processing
- database queries
- tool calls

## 8. Advantages

- Forces explicit input management.
- Helps control latency and cost.
- Encourages retrieval instead of dumping all data.
- Makes failure modes easier to reason about.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| More context vs cost | More tokens usually cost more. |
| More context vs focus | Too much irrelevant information can hurt answer quality. |
| Truncation vs completeness | Cutting context may remove important details. |
| Summarization vs accuracy | Summaries can lose nuance. |

## 10. Limitations

- Models may ignore or underuse information in long contexts.
- Important details can be lost during trimming.
- Context windows are model-specific.
- Tool outputs can unexpectedly consume large budgets.

## 11. Real-World Examples

Startup example: customer support bot includes only recent conversation and top policy chunks.

Enterprise example: contract analyzer processes long contracts section by section.

FAANG-style example: assistant infrastructure allocates context budget across instructions, memory, tools, retrieval, and answer space.

Production system: chat memory service summarizes older messages when token budget is near the limit.

## 12. Architecture Diagram

```text
[System Prompt]
[Chat History] ---> [Context Builder] -> [Budget Check] -> [LLM]
[Retrieved Docs]
[User Query]
```

## 13. Python Implementation

```python
def estimate_tokens(text: str) -> int:
    return max(1, len(text) // 4)

def build_context(parts: list[str], max_tokens: int) -> str:
    selected: list[str] = []
    used = 0

    for part in parts:
        cost = estimate_tokens(part)
        if used + cost > max_tokens:
            break
        selected.append(part)
        used += cost

    return "\n\n".join(selected)
```

Advanced context allocation:

```python
context_budget = {
    "system": 500,
    "history": 2000,
    "retrieval": 4000,
    "output": 1000,
}
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class ContextRequest(BaseModel):
    parts: list[str]
    max_input_tokens: int = 7000

@app.post("/context/build")
async def build_context_endpoint(request: ContextRequest) -> dict[str, str]:
    if request.max_input_tokens <= 0:
        raise HTTPException(status_code=400, detail="max_input_tokens must be positive")
    return {"context": build_context(request.parts, request.max_input_tokens)}
```

## 15. Database Integration

Store chat messages, summaries, document chunks, and token estimates. This lets the system rebuild context efficiently.

## 16. Production Considerations

- Always reserve output tokens.
- Never blindly concatenate all history.
- Log when context is truncated.
- Explain to users when input is too large.
- Store summaries separately from raw messages.
- Validate tool output size before reinserting into context.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Thinking the model remembers past API calls | Send required context each time |
| Intermediate | Adding entire documents to prompts | Use retrieval or chunking |
| Production | Silent truncation | Log truncation and prioritize context |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is a context window? | The maximum tokens a model can process in one request. |
| Intermediate | What counts toward it? | Instructions, history, user input, retrieved docs, tool output, and completion. |
| Advanced | How manage long conversations? | Summarize old messages, retain key facts, retrieve relevant history, and reserve output tokens. |
| Scenario | User asks about a 500-page PDF. | Ingest, chunk, embed, retrieve relevant sections, and only send useful chunks. |

## 19. System Design Discussion

Context management is a central service in production AI apps. It decides what the model sees. That makes it part of quality, security, cost, and latency control.

## 20. Hands-On Assignment

- Easy: Estimate token usage for a prompt.
- Medium: Build a context builder with a budget.
- Hard: Prioritize system prompt, latest user input, and retrieved chunks.

## 21. Mini Project

Build a chat context manager that keeps recent messages and summarizes old ones.

## 22. Production-Level Project

Build a RAG context builder that allocates budget across system instructions, user question, retrieved chunks, and answer space.

## Quiz

1. What is a context window?
2. What counts toward the context limit?
3. Does an LLM remember previous API calls automatically?
4. Why reserve output tokens?
5. Why can too much context hurt quality?
6. What is silent truncation?
7. How does RAG help context limits?
8. What is chat memory summarization?
9. Why log context truncation?
10. How would you handle a very long document?

## Knowledge Check

You should be able to design a simple context budget and explain why context is limited.

Are you ready for the next section?