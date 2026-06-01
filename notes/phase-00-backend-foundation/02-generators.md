# Generators

## 1. Problem Statement

Generators solve memory and streaming problems by producing values one at a time instead of building everything upfront.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | A generator is a function that uses `yield` to produce a sequence lazily. |
| Use When | You need streaming, pagination, or memory-efficient iteration. |
| Avoid When | You need random access to all items repeatedly. |
| Advantages | Lower memory usage and natural streaming. |
| Tradeoffs | One-pass behavior can surprise beginners. |
| Limitations | Harder debugging than simple lists. |
| Example | Reading a large log file line by line. |
| Production Example | Streaming LLM tokens to a client. |
| Interview Answer | Generators create lazy iterators that produce values on demand. |

## 3. Intermediate Explanation

Generators preserve state between yields and work well with pipelines.

## 4. Advanced Explanation

Async generators power streaming APIs where each yielded chunk becomes part of the response.

## 5. Internal Working

```text
Function call -> generator object -> next() -> yield value -> pause -> resume
```

## 6. When To Use

Use for large files, event streams, token streams, and batch processing.

## 7. When NOT To Use

Avoid when the data is small and a list is simpler.

## 8. Advantages

They reduce memory and support incremental output.

## 9. Tradeoffs

They are consumed once unless recreated.

## 10. Limitations

They do not automatically make slow operations faster.

## 11. Real-World Examples

Streaming logs, streaming chat tokens, and iterating paginated database results.

## 12. Architecture Diagram

```text
[Data Source] -> [Generator] -> [Consumer]
```

## 13. Python Implementation

```python
def chunks(text: str, size: int):
    for index in range(0, len(text), size):
        yield text[index:index + size]
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

def stream_words():
    for word in ["hello", "from", "ai"]:
        yield word + "\n"

@app.get("/stream")
def stream():
    return StreamingResponse(stream_words(), media_type="text/plain")
```

## 15. Database Integration

Use cursor-based pagination or server-side cursors for large result sets.

## 16. Production Considerations

Handle client disconnects, timeouts, and backpressure.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Expecting a generator to behave like a list | Convert only when needed with `list()` |
| Intermediate | Reusing an exhausted generator | Recreate it |
| Production | Ignoring disconnects | Add cancellation handling |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is a generator? | A lazy iterator created with `yield`. |
| Intermediate | Why use one? | To reduce memory and stream values. |
| Advanced | How does it help AI apps? | It supports token streaming and large document processing. |
| Scenario | Large file upload is slow. | Process it in chunks with generators or streaming readers. |

## 19. System Design Discussion

Generators are useful wherever an AI backend must process or return data incrementally.

## 20. Hands-On Assignment

- Easy: Yield numbers from 1 to 10.
- Medium: Yield text chunks.
- Hard: Stream generated chunks through FastAPI.

## 21. Mini Project

Build a streaming text chunk viewer.

## 22. Production-Level Project

Build a document ingestion service that processes large files chunk by chunk.

## Quiz

1. What does `yield` do?
2. Why are generators memory efficient?
3. Can a generator be reused after exhaustion?
4. What is lazy evaluation?
5. How do generators support streaming?
6. When is a list better?
7. What is an async generator?
8. What production issue happens when clients disconnect?
9. How do generators help document ingestion?
10. How would you test a generator?

## Knowledge Check

You should be able to write a generator, explain lazy iteration, and connect it to streaming AI responses.

Are you ready for the next section?