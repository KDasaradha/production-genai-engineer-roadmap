Since you want to learn this as a developer and apply it later to your GenAI roadmap, I'll teach it as a **backend-engineering bootcamp style** rather than only definitions. We'll build concepts layer by layer.

# Phase 0 — AI Backend Foundation

**Goal:** Build the backend skills that GenAI systems rely on.

Duration: **~2 weeks**

Topics:

1. AsyncIO
2. Decorators
3. Generators
4. Pydantic
5. FastAPI Dependency Injection
6. Middleware
7. PostgreSQL Optimization
8. Redis
9. Docker
10. Logging
11. Mini project: AI Streaming Chat API
12. Mini project: Background Job System

---

# Day 1: AsyncIO (Very Important)

Before GenAI, understand this:

Normal Python runs:

```python
task1()
task2()
task3()
```

Execution:

```
task1 → complete
task2 → complete
task3 → complete
```

This is **synchronous execution**.

Problem:

If `task1()` waits 5 seconds for an API response:

```python
import time

def fetch_data():
    time.sleep(5)
    return "Data"

print(fetch_data())
print("Finished")
```

Output:

```python
(wait 5 seconds)

Data
Finished
```

CPU sits idle.

---

## Async solution

Python can switch work while waiting.

```python
import asyncio

async def fetch_data():
    await asyncio.sleep(5)
    return "Data"

async def main():
    result = await fetch_data()
    print(result)

asyncio.run(main())
```

Explanation:

```python
async
```

Creates a coroutine.

```python
await
```

Pauses current task and lets Python do other work.

---

## Real-world example

Imagine:

A GenAI app receives:

* User 1 → Ask AI
* User 2 → Ask AI
* User 3 → Ask AI

Without async:

```python
User1 wait
User2 wait
User3 wait
```

With async:

```python
User1 processing
User2 processing
User3 processing
```

FastAPI heavily uses this.

---

## Concurrent execution

```python
import asyncio

async def task1():
    await asyncio.sleep(2)
    print("Task 1 done")

async def task2():
    await asyncio.sleep(1)
    print("Task 2 done")

async def main():
    await asyncio.gather(
        task1(),
        task2()
    )

asyncio.run(main())
```

Output:

```python
Task 2 done
Task 1 done
```

Why?

Because both run simultaneously.

---

# Practice Exercise 1

Create:

```python
async def get_user()
async def get_posts()
async def get_comments()
```

Rules:

* Each should sleep for random seconds

```python
await asyncio.sleep(2)
```

Run them together:

```python
asyncio.gather()
```

Expected:

```python
Fetching user...
Fetching posts...
Fetching comments...

User fetched
Comments fetched
Posts fetched
```

---

# Interview Focus

### Q1: Difference between sync and async?

**Answer:**

Synchronous executes sequentially.

Asynchronous allows execution without blocking while waiting for I/O operations.

---

### Q2: When should async be used?

Use async for:

✅ API calls
✅ Database calls
✅ AI model requests
✅ File operations

Avoid async for:

❌ Heavy CPU calculations

---

### Q3: Difference between concurrency and parallelism?

Concurrency:

```text
Task A
Task B
Task A
Task B
```

Parallelism:

```text
CPU1 → Task A
CPU2 → Task B
```

---

# Mini challenge

Build this:

```python
import asyncio

async def fetch_weather():
    pass

async def fetch_news():
    pass

async def fetch_stock():
    pass
```

Run all simultaneously and print results.

---

Tomorrow/next lesson:

**Day 2 → Decorators**

You'll learn:

* `@login_required`
* `@cache`
* `@timer`
* `@retry`
* how FastAPI internally uses decorators

These become extremely important later for Prompt Engineering + AI APIs.

---

# Python Backend Quick Notes — Interview Ready

---

# 1. Synchronous Programming (Sync)

### Description

Tasks execute **one after another**.
The next task waits until the previous task completes.

### Use When

✅ Small/simple applications
✅ Tasks depend on previous results
✅ CPU work is short

❌ Not suitable for many waiting operations (API/database calls)

### Example

```python
import time

def task1():
    print("Task1 started")
    time.sleep(2)
    print("Task1 finished")

def task2():
    print("Task2 started")
    time.sleep(2)
    print("Task2 finished")

task1()
task2()
```

### Output

```text
Task1 started
(wait 2 sec)
Task1 finished
Task2 started
(wait 2 sec)
Task2 finished
```

### Interview Answer

> Synchronous programming executes tasks sequentially where each task blocks the next task until completion.

---

# 2. Asynchronous Programming (Async)

### Description

Allows tasks to **pause while waiting** and lets other tasks execute.

Uses:

```python
async
await
```

### Use When

✅ API calls
✅ Database calls
✅ File operations
✅ AI model requests
✅ Network requests

❌ Heavy calculations

### Example

```python
import asyncio

async def task1():
    print("Task1 started")
    await asyncio.sleep(2)
    print("Task1 finished")

async def task2():
    print("Task2 started")
    await asyncio.sleep(2)
    print("Task2 finished")

async def main():
    await task1()
    await task2()

asyncio.run(main())
```

### Interview Answer

> Async programming avoids blocking while waiting for I/O operations.

---

# 3. Concurrency

### Description

Multiple tasks progress **at the same time by switching between tasks**.

Single CPU can manage multiple tasks.

### Use When

✅ Multiple users hitting APIs
✅ Chat applications
✅ Streaming systems
✅ GenAI APIs

### Example

```python
import asyncio

async def task1():
    await asyncio.sleep(2)
    print("Task1 done")

async def task2():
    await asyncio.sleep(1)
    print("Task2 done")

async def main():
    await asyncio.gather(
        task1(),
        task2()
    )

asyncio.run(main())
```

### Output

```text
Task2 done
Task1 done
```

### Interview Answer

> Concurrency means handling multiple tasks during overlapping time periods.

---

# 4. Parallelism

### Description

Multiple tasks execute **literally at the same time using multiple CPU cores**.

### Use When

✅ Image processing
✅ Machine learning computation
✅ Video processing
✅ Heavy calculations

### Example

```python
from multiprocessing import Process
import time

def task(name):
    print(f"{name} started")
    time.sleep(2)
    print(f"{name} finished")

p1 = Process(target=task,args=("Task1",))
p2 = Process(target=task,args=("Task2",))

p1.start()
p2.start()

p1.join()
p2.join()
```

### Interview Answer

> Parallelism executes multiple tasks simultaneously using multiple CPU cores.

---

# 5. CPU Bound Tasks

### Description

Tasks mainly spend time **using CPU power**.

The CPU becomes the bottleneck.

### Examples

* Image processing
* Video encoding
* Encryption
* Machine learning training
* Large mathematical calculations

### Example

```python
def calculate():

    total = 0

    for i in range(100000000):
        total += i

    print(total)

calculate()
```

### Use

✅ Multiprocessing
✅ Parallelism

❌ Async

### Interview Answer

> CPU-bound tasks spend most of their time performing calculations.

---

# 6. I/O Bound Tasks

### Description

Tasks spend most time **waiting for external resources**.

CPU stays idle while waiting.

### Examples

* API calls
* Database queries
* Reading files
* AI API requests
* Network operations

### Example

```python
import requests

response = requests.get(
    "https://jsonplaceholder.typicode.com/posts"
)

print(response.json())
```

### Use

✅ AsyncIO
✅ Threading

❌ Multiprocessing

### Interview Answer

> I/O-bound tasks spend most of their time waiting for external operations.

---

# Scenario Cheat Sheet

| Scenario                        |             Use |
| ------------------------------- | --------------: |
| Calling external APIs           |         AsyncIO |
| Database queries                |         AsyncIO |
| AI chatbot requests             |         AsyncIO |
| Multiple users accessing API    |     Concurrency |
| Background jobs                 | AsyncIO + Queue |
| Video processing                | Multiprocessing |
| Image processing                | Multiprocessing |
| ML model training               |     Parallelism |
| Heavy mathematical calculations | Multiprocessing |
| Small scripts                   |            Sync |

---

# One-Line Memory Trick

```text
Sync → One by one

Async → Don't wait while waiting

Concurrency → Handle many tasks together

Parallelism → Run many tasks together

CPU Bound → Heavy calculation work

I/O Bound → Waiting work
```

---

# Most Asked Interview Question

**Q: Async vs Concurrency vs Parallelism**

**Answer:**

* Async → Non-blocking execution
* Concurrency → Multiple tasks progress together
* Parallelism → Multiple tasks execute simultaneously using multiple CPUs
---

# Python Backend Quick Notes — Threading & Multiprocessing (Interview Ready)

---

# 1. Thread

### Description

A **thread** is the smallest unit of execution inside a process.

Multiple threads **share the same memory space**.

```text
Process
   |
   ├── Thread 1
   ├── Thread 2
   └── Thread 3
```

### Use When

✅ I/O-bound tasks
✅ API calls
✅ File reading/writing
✅ Database operations

❌ Heavy CPU calculations

### Example

```python
import threading
import time

def task():
    print("Task started")
    time.sleep(2)
    print("Task finished")

t = threading.Thread(target=task)

t.start()
t.join()

print("Main finished")
```

### Interview Answer

> A thread is a lightweight execution unit inside a process that shares memory with other threads.

---

# 2. Process

### Description

A **process** is an independent running program with its own memory space.

```text
OS
   |
   ├── Process 1
   ├── Process 2
   └── Process 3
```

Each process:

* Has separate memory
* Runs independently
* Cannot directly access another process's memory

### Use When

✅ Heavy calculations
✅ Machine learning training
✅ Image processing
✅ Video processing

### Example

```python
from multiprocessing import Process
import time

def task():
    print("Task started")
    time.sleep(2)
    print("Task finished")

p = Process(target=task)

p.start()
p.join()

print("Main finished")
```

### Interview Answer

> A process is an independent program execution instance with separate memory space.

---

# 3. Multithreading

### Description

Multiple threads execute inside a single process.

Threads share:

* Memory
* Variables
* Resources

```text
Process
   |
   ├── Thread A
   ├── Thread B
   └── Thread C
```

### Use When

✅ Multiple API requests
✅ Download managers
✅ Chat servers
✅ Reading multiple files

### Example

```python
import threading
import time

def task(name):

    print(f"{name} started")

    time.sleep(2)

    print(f"{name} finished")


t1 = threading.Thread(
    target=task,
    args=("Task1",)
)

t2 = threading.Thread(
    target=task,
    args=("Task2",)
)

t1.start()
t2.start()

t1.join()
t2.join()
```

### Output

```text
Task1 started
Task2 started

(wait)

Task1 finished
Task2 finished
```

### Interview Answer

> Multithreading runs multiple threads inside one process for better I/O performance.

---

# 4. Multiprocessing

### Description

Multiple processes execute simultaneously.

Each process has:

* Separate memory
* Separate resources

```text
OS
   |
   ├── Process A
   ├── Process B
   └── Process C
```

### Use When

✅ CPU-intensive tasks
✅ AI model training
✅ Video rendering
✅ Data processing

### Example

```python
from multiprocessing import Process
import time

def task(name):

    print(f"{name} started")

    time.sleep(2)

    print(f"{name} finished")


p1 = Process(
    target=task,
    args=("Task1",)
)

p2 = Process(
    target=task,
    args=("Task2",)
)

p1.start()
p2.start()

p1.join()
p2.join()
```

### Interview Answer

> Multiprocessing executes multiple processes independently using multiple CPU cores.

---

# Thread vs Process

| Feature       |          Thread |             Process |
| ------------- | --------------: | ------------------: |
| Memory        |          Shared |            Separate |
| Speed         | Faster creation |     Slower creation |
| Communication |            Easy |              Harder |
| CPU usage     |  Limited by GIL | Uses multiple cores |
| Best for      |       I/O tasks |           CPU tasks |

---

# Multithreading vs Multiprocessing

| Feature        |               Multithreading |         Multiprocessing |
| -------------- | ---------------------------: | ----------------------: |
| Execution unit |                      Threads |               Processes |
| Memory         |                       Shared |                Separate |
| CPU usage      | Single core limitation (GIL) |          Multiple cores |
| Best for       |                    I/O-bound |               CPU-bound |
| Performance    |     Better for waiting tasks | Better for calculations |

---

# Python GIL (Very Common Interview Question)

### Description

GIL = **Global Interpreter Lock**

Python allows:

```text
Only one thread executes Python bytecode at a time
```

Because of GIL:

* Multithreading is not efficient for CPU-heavy work
* Multithreading works well for I/O waiting tasks

### Example

Bad:

```python
# CPU heavy work

for i in range(100000000):
    total += i
```

Good:

```python
# API requests

requests.get(url)
```

### Interview Answer

> GIL allows only one thread to execute Python bytecode at a time, which limits CPU-bound multithreading performance.

---

# Scenario Cheat Sheet

| Scenario                  |                      Use |
| ------------------------- | -----------------------: |
| Multiple API requests     |           Multithreading |
| Database queries          |           Multithreading |
| File downloads            |           Multithreading |
| Chat server               |           Multithreading |
| AI API calls              | AsyncIO / Multithreading |
| Image processing          |          Multiprocessing |
| Video rendering           |          Multiprocessing |
| Machine learning training |          Multiprocessing |
| Heavy calculations        |          Multiprocessing |

---

# One-Line Memory Trick

```text
Thread → Small worker inside process

Process → Independent running program

Multithreading → Multiple workers sharing same room

Multiprocessing → Multiple workers in separate rooms

I/O-bound → Threading

CPU-bound → Multiprocessing
```

---

# Most Asked Interview Question

**Q: Thread vs Process vs Async**

**Answer:**

* Thread → Lightweight execution inside process, shared memory
* Process → Independent execution with separate memory
* Async → Non-blocking execution for waiting operations

```text
Waiting work → Async/Threading

Heavy computation → Multiprocessing
```

---

# Python Backend Quick Notes — Tradeoffs & Limitations (Interview Ready)

---

# 1. Synchronous Programming (Sync)

### Tradeoffs

✅ Simple to write
✅ Easy debugging
✅ Easy to understand flow

❌ Poor performance with waiting tasks
❌ Blocks entire execution
❌ Low scalability

### Limitations

* One task waits for another
* Wastes CPU during I/O waiting
* Bad for handling many users

### Example Problem

```python
import time

def api_call():
    time.sleep(5)

api_call()
print("Done")
```

Problem:

```text
Everything waits for 5 seconds
```

---

# 2. Asynchronous Programming (Async)

### Tradeoffs

✅ Handles many I/O requests efficiently
✅ Better resource utilization
✅ High scalability

❌ More difficult debugging
❌ Requires understanding event loop
❌ Can become hard to maintain

### Limitations

* Not useful for CPU-heavy work
* Blocking code can break async performance
* Mixing sync and async can create issues

### Bad Example

```python
import asyncio
import time

async def task():
    time.sleep(5)

asyncio.run(task())
```

Problem:

```text
time.sleep() blocks event loop
```

Correct:

```python
await asyncio.sleep(5)
```

---

# 3. Concurrency

### Tradeoffs

✅ Better user responsiveness
✅ Improves resource usage

❌ Adds complexity
❌ Hard debugging

### Limitations

* Race conditions
* Deadlocks
* Shared resource issues

### Example Problem

```python
counter = 0

def increment():
    global counter
    counter += 1
```

Problem:

```text
Multiple tasks may modify same data
```

---

# 4. Parallelism

### Tradeoffs

✅ Maximum CPU utilization
✅ Faster heavy computations

❌ High memory usage
❌ Process creation overhead

### Limitations

* Inter-process communication cost
* Context switching overhead
* Increased system resource consumption

---

# 5. CPU-bound Tasks

### Tradeoffs

✅ Fast calculations using multiple cores

❌ High CPU consumption

### Limitations

* Async provides little benefit
* Multithreading affected by GIL
* Can slow the whole system

Example:

```python
for i in range(100000000):
    total += i
```

---

# 6. I/O-bound Tasks

### Tradeoffs

✅ Great with Async/Threading

❌ Waiting operations dominate time

### Limitations

* Network delays still exist
* External services can become bottlenecks

Example:

```python
requests.get(url)
```

---

# 7. Thread

### Tradeoffs

✅ Lightweight
✅ Fast creation
✅ Shared memory

❌ Shared memory creates risk

### Limitations

* Race conditions
* Deadlocks
* GIL restriction
* Not ideal for CPU-heavy work

Example issue:

```python
counter += 1
```

Problem:

```text
Two threads updating same value simultaneously
```

---

# 8. Process

### Tradeoffs

✅ True parallel execution
✅ Separate memory improves safety

❌ More memory usage
❌ Higher startup cost

### Limitations

* Slower communication
* Larger system overhead

---

# 9. Multithreading

### Tradeoffs

✅ Better for waiting operations
✅ Lower memory usage

❌ GIL limitation

### Limitations

* Race conditions
* Shared state complexity
* Deadlocks

Example issue:

```python
balance -= 100
```

Problem:

```text
Multiple threads updating same balance
```

---

# 10. Multiprocessing

### Tradeoffs

✅ Uses multiple CPU cores
✅ Avoids GIL

❌ Higher memory consumption

### Limitations

* Process startup overhead
* Harder communication
* Serialization cost

Example issue:

```python
large_data = [1]*100000000
```

Problem:

```text
Data copied across processes consumes memory
```

---

# Race Condition

### Description

Multiple threads/processes modify shared data simultaneously.

Example:

```python
import threading

counter = 0

def increment():
    global counter

    for i in range(100000):
        counter += 1

t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)

t1.start()
t2.start()

t1.join()
t2.join()

print(counter)
```

Expected:

```text
200000
```

Possible output:

```text
174523
```

Problem:

```text
Shared memory conflict
```

Solution:

```python
threading.Lock()
```

---

# Deadlock

### Description

Two tasks wait for each other forever.

Example:

```python
Thread1 → waiting for Resource2
Thread2 → waiting for Resource1
```

Result:

```text
Infinite waiting
```

Solution:

* Lock ordering
* Timeouts
* Minimize shared resources

---

# Final Interview Cheat Sheet

| Situation           |     Best Choice |           Avoid |
| ------------------- | --------------: | --------------: |
| Small scripts       |            Sync |           Async |
| API requests        |           Async | Multiprocessing |
| Database calls      |           Async | Multiprocessing |
| File downloads      |       Threading | Multiprocessing |
| Multiple users      |     Concurrency |            Sync |
| Image processing    | Multiprocessing |           Async |
| ML training         | Multiprocessing |       Threading |
| Heavy calculations  | Multiprocessing |           Async |
| Shared data updates |           Locks |   Direct access |

---

# One-Line Memory Trick

```text
Sync → Simple but blocks

Async → Efficient but complex

Thread → Lightweight but GIL limited

Process → Powerful but expensive

Multithreading → Good for I/O

Multiprocessing → Good for CPU

Concurrency → Manage many tasks

Parallelism → Execute many tasks

Race Condition → Multiple updates collide

Deadlock → Tasks wait forever
```
---
