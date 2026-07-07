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
