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

---

# Generators — Beginner to Advanced (Interview Ready)

Generators are one of the most frequently asked Python interview topics because they test your understanding of:

* Memory management
* Iteration
* Lazy loading
* Performance optimization

They are heavily used in:

* FastAPI Streaming Responses
* Large file processing
* Data pipelines
* AI/ML data loading
* Log processing

---

# 1. What is a Generator?

A generator is a special function that produces values **one at a time** instead of creating all values at once.

Normal function:

```python
def numbers():
    return [1, 2, 3]
```

Generator:

```python
def numbers():
    yield 1
    yield 2
    yield 3
```

The keyword `yield` makes a function a generator.

---

# 2. Why Do We Need Generators?

Suppose you need:

```python
1 million records
```

### Using List

```python
numbers = [x for x in range(1000000)]
```

Memory:

```text
Stores all 1 million values in RAM
```

---

### Using Generator

```python
numbers = (x for x in range(1000000))
```

Memory:

```text
Creates values only when needed
```

This is called **Lazy Evaluation**.

---

# 3. How Normal Functions Work

```python
def test():
    print("A")
    print("B")
    return "Done"
```

Execution:

```python
test()
```

Output:

```text
A
B
Done
```

Function starts and ends once.

---

# 4. How Generator Functions Work

```python
def test():
    print("A")
    yield 1

    print("B")
    yield 2
```

Calling:

```python
g = test()
```

Nothing executes yet.

Output:

```text
Nothing
```

Why?

Because generators are lazy.

---

# 5. next() Function

```python
g = test()

print(next(g))
```

Output:

```text
A
1
```

Generator pauses at:

```python
yield 1
```

---

Call again:

```python
print(next(g))
```

Output:

```text
B
2
```

Generator resumes from where it stopped.

---

# 6. Internal Working

Generator execution flow:

```text
Create Generator
      ↓
next()
      ↓
Run until yield
      ↓
Pause
      ↓
Store State
      ↓
next()
      ↓
Resume
```

This pause/resume behavior is what makes generators powerful.

---

# 7. Generator State Preservation

Example:

```python
def counter():

    count = 1

    while True:
        yield count
        count += 1
```

Usage:

```python
c = counter()

print(next(c))
print(next(c))
print(next(c))
```

Output:

```text
1
2
3
```

Notice:

```python
count
```

is preserved between calls.

Normal functions cannot do this automatically.

---

# 8. Generator vs Return

### Return

```python
def test():
    return 1
```

Execution:

```text
Returns value
Function ends forever
```

---

### Yield

```python
def test():
    yield 1
```

Execution:

```text
Returns value
Pauses function
Can resume later
```

---

# 9. Generator Object

```python
def numbers():
    yield 1
```

```python
g = numbers()
```

Type:

```python
print(type(g))
```

Output:

```python
<class 'generator'>
```

---

# 10. Iterating Through Generator

Instead of:

```python
next(g)
next(g)
next(g)
```

Use:

```python
for value in numbers():
    print(value)
```

Output:

```text
1
2
3
```

---

# 11. StopIteration Exception

```python
def numbers():
    yield 1
```

```python
g = numbers()

next(g)
next(g)
```

Output:

```text
StopIteration
```

Reason:

No more values available.

---

# 12. Generator Expression

List comprehension:

```python
squares = [x*x for x in range(5)]
```

Output:

```python
[0,1,4,9,16]
```

Stored fully in memory.

---

Generator expression:

```python
squares = (x*x for x in range(5))
```

Output:

```python
<generator object>
```

Generated lazily.

---

# 13. Memory Comparison

List:

```python
numbers = [x for x in range(1000000)]
```

Stores:

```text
1,000,000 values
```

---

Generator:

```python
numbers = (x for x in range(1000000))
```

Stores:

```text
Current state only
```

Huge memory savings.

---

# 14. Real World Example — Large File Processing

Bad:

```python
with open("huge.log") as f:
    data = f.readlines()
```

Loads entire file.

---

Better:

```python
with open("huge.log") as f:

    for line in f:
        process(line)
```

Python file objects are generators/iterators.

Only one line loaded at a time.

---

# 15. Real World Example — Streaming Data

```python
def stream_numbers():

    for i in range(1000000):
        yield str(i)
```

FastAPI:

```python
from fastapi.responses import StreamingResponse

@app.get("/stream")
def stream():

    return StreamingResponse(
        stream_numbers()
    )
```

Data sent gradually.

Memory efficient.

---

# 16. Infinite Generators

```python
def infinite():

    num = 1

    while True:
        yield num
        num += 1
```

Usage:

```python
g = infinite()

print(next(g))
print(next(g))
print(next(g))
```

Output:

```text
1
2
3
```

Can theoretically run forever.

---

# 17. Generator Pipeline

Example:

```python
def numbers():
    for i in range(10):
        yield i
```

---

Filter:

```python
def even(nums):

    for n in nums:
        if n % 2 == 0:
            yield n
```

---

Usage:

```python
result = even(numbers())

for x in result:
    print(x)
```

Output:

```text
0
2
4
6
8
```

Very common in ETL pipelines.

---

# 18. Yield From

Without:

```python
def combined():

    for x in range(3):
        yield x

    for x in range(3,6):
        yield x
```

---

With:

```python
def combined():

    yield from range(3)

    yield from range(3,6)
```

Cleaner.

---

# 19. Generator vs Iterator

Interview favorite.

### Iterator

Object implementing:

```python
__iter__()
__next__()
```

Example:

```python
iter([1,2,3])
```

---

### Generator

Special type of iterator created using:

```python
yield
```

Every generator is an iterator.

Not every iterator is a generator.

---

# 20. When to Use Generators

Use when:

✅ Large datasets

✅ Streaming

✅ File processing

✅ API responses

✅ Infinite sequences

✅ ETL pipelines

✅ AI data loaders

---

Avoid when:

❌ Need random access

```python
data[500]
```

❌ Need multiple passes

```python
for x in gen:
    ...

for x in gen:
    ...
```

Second loop may be empty because generator is exhausted.

---

# Generator Lifecycle

```text
Create Generator
       ↓
next()
       ↓
Run Until yield
       ↓
Pause
       ↓
Store State
       ↓
next()
       ↓
Resume
       ↓
Complete
       ↓
StopIteration
```

---

# Common Interview Questions

### Q1. What is a Generator?

**Answer**

A generator is a special function that uses `yield` to produce values lazily, one at a time, instead of storing all values in memory.

---

### Q2. Difference Between Return and Yield?

| Return             | Yield           |
| ------------------ | --------------- |
| Ends function      | Pauses function |
| One value          | Multiple values |
| No state preserved | State preserved |

---

### Q3. Why Are Generators Memory Efficient?

**Answer**

Because they generate values on demand instead of storing the entire dataset in memory.

---

### Q4. What Happens After Yield?

**Answer**

Execution pauses and the function state is saved. The next call resumes from the same point.

---

### Q5. Difference Between Generator and Iterator?

**Answer**

Generators are a simpler way to create iterators using `yield`. Every generator is an iterator, but not every iterator is a generator.

---

# Interview Answer (2-Minute Version)

> A generator is a special Python function that uses `yield` instead of `return`. Unlike normal functions, generators produce values one at a time and preserve their execution state between calls. This makes them memory efficient because they don't load all data into memory at once. They are commonly used for large datasets, file processing, streaming responses, and ETL pipelines. Every generator is an iterator and supports lazy evaluation through the `next()` function.

---

## What You Should Master Before Moving On

* `yield`
* `next()`
* Generator expressions
* `yield from`
* StopIteration
* Generator vs Iterator
* Memory benefits
* FastAPI streaming use cases

After Generators, the next topic should be **AsyncIO**, because AsyncIO is the backbone of FastAPI and one of the highest-priority topics in Python backend interviews.

---

# Day 3: Generators + `yield` + Memory Optimization + AI Streaming

Generators become very important in AI systems because LLMs and APIs often **stream data gradually** instead of returning everything at once.

Examples:

* AI chat streaming (`ChatGPT typing effect`)
* Reading huge files
* Database records
* Background processing
* Log streaming
* Data pipelines

---

# Definition

A generator is a function that **returns values one at a time using `yield`**, instead of returning everything at once.

Normal function:

```python
def numbers():
    return [1,2,3,4,5]

print(numbers())
```

Output:

```python
[1,2,3,4,5]
```

Entire list is created in memory immediately.

---

Generator version:

```python
def numbers():

    yield 1
    yield 2
    yield 3
    yield 4
    yield 5


x = numbers()

print(next(x))
print(next(x))
print(next(x))
```

Output:

```text
1
2
3
```

---

# How `yield` works internally

Normal function:

```python
def demo():

    print("A")
    print("B")
    print("C")
```

Execution:

```text
A
B
C
```

Generator:

```python
def demo():

    print("A")
    yield 1

    print("B")
    yield 2

    print("C")
    yield 3


g=demo()

print(next(g))
print(next(g))
print(next(g))
```

Output:

```text
A
1

B
2

C
3
```

Notice:

Function **pauses and remembers state**.

---

# Visual flow

```text
next()

↓

Function starts

↓

yield value

↓

Paused

↓

next()

↓

Resume from previous location

↓

yield value

↓

Paused
```

---

# Generator vs Return

| Feature                     | Return | Yield |
| --------------------------- | -----: | ----: |
| Returns all values          |      ✅ |     ❌ |
| Stores all data in memory   |      ✅ |     ❌ |
| Returns one value at a time |      ❌ |     ✅ |
| Supports lazy loading       |      ❌ |     ✅ |
| Good for huge datasets      |      ❌ |     ✅ |

---

# Memory problem example

Bad approach:

```python
def million():

    nums=[]

    for i in range(1000000):
        nums.append(i)

    return nums
```

This stores:

```text
1,000,000 values in RAM
```

Generator solution:

```python
def million():

    for i in range(1000000):
        yield i
```

Memory usage becomes very small because values are produced only when needed.

---

# Looping through generators

```python
def numbers():

    for i in range(5):
        yield i


for x in numbers():
    print(x)
```

Output:

```text
0
1
2
3
4
```

Python automatically calls:

```python
next()
```

internally.

---

# Generator expression

List comprehension:

```python
nums=[x*x for x in range(5)]

print(nums)
```

Output:

```python
[0,1,4,9,16]
```

Generator expression:

```python
nums=(x*x for x in range(5))

print(nums)
```

Output:

```python
<generator object ...>
```

Retrieve values:

```python
for i in nums:
    print(i)
```

Output:

```text
0
1
4
9
16
```

---

# Real AI Example — Token streaming

Without streaming:

```python
def ai_response():

    return "Hello I am your AI assistant"
```

User waits for complete response.

---

Streaming:

```python
import time

def ai_response():

    words=["Hello","I","am","your","AI","assistant"]

    for word in words:

        time.sleep(1)

        yield word


for token in ai_response():

    print(token)
```

Output:

```text
Hello
I
am
your
AI
assistant
```

This is similar to:

```text
ChatGPT typing...
```

---

# FastAPI streaming example

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import time

app=FastAPI()


def generate():

    for i in range(5):

        time.sleep(1)

        yield f"Message {i}\n"


@app.get("/stream")
async def stream():

    return StreamingResponse(generate())
```

Open:

```text
http://localhost:8000/stream
```

Instead of waiting for all data:

```text
Message 0
Message 1
Message 2
Message 3
```

arrives gradually.

---

# Async generators

Normal generator:

```python
def generate():

    yield 1
```

Async generator:

```python
import asyncio

async def generate():

    for i in range(5):

        await asyncio.sleep(1)

        yield i
```

Usage:

```python
async for x in generate():
    print(x)
```

---

# Practice Exercise

Build:

### 1. Log streamer

```python
def logs():

    yield "Server started"

    yield "Database connected"

    yield "Request received"
```

---

### 2. Fake AI chat streaming

Expected:

```text
AI:
Hello
How
can
I
help
you?
```

---

### 3. Large CSV reader

Instead of:

```python
rows=read_all()
```

Build:

```python
def read_csv():
    yield row
```

---

# Interview Focus

### Q1: What is a generator?

A generator is a function that returns values one at a time using `yield`.

---

### Q2: Why use generators?

* Lower memory usage
* Lazy loading
* Streaming data
* Large datasets

---

### Q3: Difference between `return` and `yield`?

`return` ends execution and returns everything.

`yield` pauses execution and returns one value at a time.

---

### Q4: Where are generators used in backend systems?

* Streaming APIs
* File processing
* Database records
* Event processing
* AI token streaming

---

# Mini Project Task

Build:

```python
@app.get("/chat")
async def chat():
    return StreamingResponse(generate_ai_response())
```

where:

```python
generate_ai_response()
```

streams words one by one.

Next lesson:

**Day 4 → Pydantic + validation + request/response schemas + production FastAPI patterns**
