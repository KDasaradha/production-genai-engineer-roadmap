## 1) Synchronous Programming (Sync)

### Definition

A programming model where tasks execute **one after another** in sequence. The next task starts only after the current one finishes.

### Use When

* Tasks depend on previous results
* Small/simple applications
* CPU work is short
* Predictable execution order matters

### Avoid When

* Heavy network calls
* File operations
* Multiple external APIs
* High-concurrency systems

### Advantages / Tradeoffs

**Advantages**

* Easy to read
* Easy debugging
* Predictable flow

**Tradeoffs**

* Slow if waiting on I/O
* Resources stay idle while waiting

### Limitations

* Cannot efficiently utilize waiting time
* Poor scalability under many requests

### Failure Scenario / Pitfall

Application freezes because one API call takes 10 seconds.

### Common Problems / Fixes

| Problem             | Fix                      |
| ------------------- | ------------------------ |
| Slow response time  | Use async                |
| Blocking operations | Move to background tasks |
| Low throughput      | Introduce concurrency    |

### Example

```python
import time

def task1():
    print("Task1 started")
    time.sleep(3)
    print("Task1 completed")

def task2():
    print("Task2 started")
    time.sleep(2)
    print("Task2 completed")

task1()
task2()
```

Output:

```text
Task1 started
(wait 3s)
Task1 completed
Task2 started
(wait 2s)
Task2 completed
```

Total ≈ 5 seconds

### Interview Answer

> Synchronous programming executes tasks sequentially where each operation blocks the next one until completion. It is simple and predictable but inefficient for I/O-heavy applications because resources stay idle while waiting.

---

## 2) Asynchronous Programming (Async)

### Definition

A programming model where tasks can **start and pause while waiting**, allowing other tasks to execute.

### Use When

* API requests
* Database operations
* File I/O
* Chat systems
* Web servers

### Avoid When

* Heavy CPU calculations
* Simple scripts
* Tasks requiring strict sequence

### Advantages / Tradeoffs

**Advantages**

* Better throughput
* Efficient resource usage
* Handles many requests

**Tradeoffs**

* More complex
* Harder debugging

### Limitations

* Not ideal for CPU-intensive work
* Requires async-compatible libraries

### Failure Scenario / Pitfall

Calling blocking code inside async:

```python
async def get_data():
    time.sleep(5)   # Wrong
```

Blocks the event loop.

### Common Problems / Fixes

| Problem               | Fix                                  |
| --------------------- | ------------------------------------ |
| Event loop blocked    | Replace with `await asyncio.sleep()` |
| Mixed sync/async code | Use async-compatible libraries       |
| Forgotten await       | Add `await`                          |

### Example

```python
import asyncio

async def task1():
    print("Task1 started")
    await asyncio.sleep(3)
    print("Task1 completed")

async def task2():
    print("Task2 started")
    await asyncio.sleep(2)
    print("Task2 completed")

async def main():
    await asyncio.gather(
        task1(),
        task2()
    )

asyncio.run(main())
```

Total ≈ 3 seconds

### Interview Answer

> Asynchronous programming allows tasks to pause during waiting operations and lets other tasks execute in the meantime. It is highly effective for I/O-bound applications and improves throughput.

---

## 3) Concurrency

### Definition

Multiple tasks **make progress during the same period**, but not necessarily at the same instant.

Think:

```text
Task A → Task B → Task A → Task B
```

### Use When

* Web servers
* Request handling
* Chat applications
* I/O-heavy workloads

### Avoid When

* Single simple task
* Heavy CPU computation only

### Advantages / Tradeoffs

**Advantages**

* Better responsiveness
* Improved resource usage

**Tradeoffs**

* Synchronization complexity

### Limitations

* Can introduce race conditions
* Hard debugging

### Failure Scenario / Pitfall

Shared variable modification:

```python
counter += 1
```

Multiple threads may corrupt data.

### Common Problems / Fixes

| Problem             | Fix                    |
| ------------------- | ---------------------- |
| Race condition      | Locks                  |
| Shared state issues | Thread-safe structures |
| Deadlock            | Proper lock ordering   |

### Example

```python
import threading

def task():
    print("Running")

t1 = threading.Thread(target=task)
t2 = threading.Thread(target=task)

t1.start()
t2.start()
```

### Interview Answer

> Concurrency means multiple tasks progress during overlapping time periods. It improves responsiveness and resource utilization but introduces synchronization challenges.

---

## 4) Parallelism

### Definition

Multiple tasks execute **at exactly the same time** using multiple CPU cores.

Think:

```text
Core1 → Task A
Core2 → Task B
Core3 → Task C
```

### Use When

* Image processing
* Machine learning
* Scientific calculations
* Heavy CPU tasks

### Avoid When

* Lightweight tasks
* Mostly I/O work

### Advantages / Tradeoffs

**Advantages**

* Faster execution
* Better CPU utilization

**Tradeoffs**

* Higher memory usage
* Process management overhead

### Limitations

* Requires multiple cores
* Communication cost between processes

### Failure Scenario / Pitfall

Creating too many processes:

```python
for i in range(1000):
    Process(...)
```

System overhead becomes high.

### Common Problems / Fixes

| Problem            | Fix              |
| ------------------ | ---------------- |
| Too many processes | Limit pool size  |
| High memory usage  | Use worker pools |
| Process overhead   | Batch tasks      |

### Example

```python
from multiprocessing import Process

def task():
    print("Processing")

p1 = Process(target=task)
p2 = Process(target=task)

p1.start()
p2.start()

p1.join()
p2.join()
```

### Interview Answer

> Parallelism means executing multiple tasks simultaneously on multiple CPU cores. It is mainly used for CPU-bound workloads to reduce total execution time.

Next batch: **CPU-Bound, I/O-Bound, Thread, Process**.

## 5) CPU-Bound Tasks

### Definition

Tasks where the **CPU is the bottleneck**. The program spends most of its time performing calculations rather than waiting.

Examples:

* Image processing
* Video encoding
* Machine learning training
* Data analysis
* Mathematical computations

### Use When

* Heavy calculations
* Large datasets
* Scientific computation
* Encryption/compression

### Avoid When

* Mostly waiting for network/database responses
* File reads/writes dominate execution

### Advantages / Tradeoffs

**Advantages**

* Can benefit greatly from multiple CPU cores
* Good candidate for multiprocessing

**Tradeoffs**

* High CPU consumption
* Can slow the entire system

### Limitations

* Python threads are limited by GIL for CPU work
* Requires more processing resources

### Failure Scenario / Pitfall

Using threads for CPU-heavy work:

```python
import threading

def calculate():
    for _ in range(10**8):
        pass

t1 = threading.Thread(target=calculate)
t2 = threading.Thread(target=calculate)

t1.start()
t2.start()
```

Threads may not significantly improve speed due to GIL.

### Common Problems / Fixes

| Problem             | Fix                 |
| ------------------- | ------------------- |
| Slow processing     | Use multiprocessing |
| CPU overload        | Batch tasks         |
| Thread inefficiency | Use process pools   |

### Example

```python
def factorial(n):
    result=1

    for i in range(1,n+1):
        result*=i

    return result

print(factorial(100000))
```

### Interview Answer

> CPU-bound tasks spend most of their execution time doing calculations rather than waiting for external resources. They benefit more from multiprocessing than multithreading in Python.

---

## 6) I/O-Bound Tasks

### Definition

Tasks where execution spends most of its time **waiting for external operations**.

Examples:

* API calls
* Database queries
* File operations
* Reading from network

### Use When

* Web applications
* Microservices
* Database systems
* Chat applications

### Avoid When

* Heavy calculations dominate

### Advantages / Tradeoffs

**Advantages**

* Works well with async
* High throughput possible
* Efficient resource use

**Tradeoffs**

* Complex async debugging

### Limitations

* Depends on external system speed
* Network latency still exists

### Failure Scenario / Pitfall

Blocking I/O inside async:

```python
async def fetch():
    requests.get("https://api.example.com")
```

`requests` blocks execution.

### Common Problems / Fixes

| Problem               | Fix                    |
| --------------------- | ---------------------- |
| Blocking libraries    | Use async versions     |
| Slow external service | Add timeout            |
| Too many requests     | Use connection pooling |

### Example

```python
import requests

response = requests.get(
    "https://jsonplaceholder.typicode.com/posts"
)

print(response.status_code)
```

### Interview Answer

> I/O-bound tasks spend most of their time waiting for external operations like databases, APIs, or files. Async programming and multithreading improve performance for these workloads.

---

## 7) Thread

### Definition

A thread is the **smallest unit of execution within a process**.

Multiple threads:

* Share memory
* Share variables
* Run independently

Structure:

```text
Process
 ├── Thread1
 ├── Thread2
 └── Thread3
```

### Use When

* I/O-heavy tasks
* Background jobs
* Concurrent request handling

### Avoid When

* CPU-intensive work in Python

### Advantages / Tradeoffs

**Advantages**

* Lightweight
* Shared memory
* Faster creation

**Tradeoffs**

* Shared memory creates synchronization issues

### Limitations

* Python GIL limits CPU parallelism

### Failure Scenario / Pitfall

Race condition:

```python
counter += 1
```

Multiple threads update simultaneously.

### Common Problems / Fixes

| Problem                | Fix                    |
| ---------------------- | ---------------------- |
| Race condition         | Locks                  |
| Shared data corruption | Thread-safe structures |
| Resource leaks         | Proper cleanup         |

### Example

```python
import threading

def task():
    print("Running thread")

thread = threading.Thread(
    target=task
)

thread.start()
thread.join()
```

### Interview Answer

> A thread is the smallest execution unit inside a process. Threads share memory and are lightweight, making them useful for I/O-bound concurrency.

---

## 8) Process

### Definition

A process is an **independent program instance with its own memory space**.

Structure:

```text
OS
 ├── Process1
 ├── Process2
 └── Process3
```

Unlike threads:

* Separate memory
* Independent execution
* Better isolation

### Use When

* CPU-intensive tasks
* True parallel execution
* Large computations

### Avoid When

* Small lightweight tasks
* Frequent communication required

### Advantages / Tradeoffs

**Advantages**

* True parallelism
* Better isolation
* Avoids GIL issues

**Tradeoffs**

* Higher memory usage
* Communication overhead

### Limitations

* Expensive to create
* Data sharing is harder

### Failure Scenario / Pitfall

Creating excessive processes:

```python
for i in range(1000):
    Process(target=task)
```

Huge overhead and memory consumption.

### Common Problems / Fixes

| Problem            | Fix                |
| ------------------ | ------------------ |
| High memory usage  | Process pool       |
| Slow communication | Shared queues      |
| Too many processes | Limit worker count |

### Example

```python
from multiprocessing import Process

def task():
    print("Running process")

p = Process(target=task)

p.start()
p.join()
```

### Interview Answer

> A process is an independent execution unit with separate memory space. Processes provide true parallelism and are commonly used for CPU-bound workloads in Python.

Next batch: **Multithreading, Multiprocessing, Python GIL, Race Condition**.

## 9) Multithreading

### Definition

Multithreading is running **multiple threads inside a single process** to perform tasks concurrently.

Structure:

```text
Process
 ├── Thread1
 ├── Thread2
 └── Thread3
```

Threads:

* Share memory
* Share resources
* Execute independently

### Use When

* API calls
* File operations
* Database queries
* Background tasks
* I/O-bound workloads

### Avoid When

* Heavy CPU computation in Python
* Strong isolation is required

### Advantages / Tradeoffs

**Advantages**

* Lightweight
* Fast thread creation
* Shared memory makes communication easy
* Improves responsiveness

**Tradeoffs**

* Shared memory introduces synchronization problems
* Python GIL limits CPU parallelism

### Limitations

* Cannot fully utilize multiple cores for CPU-heavy work (Python)
* Debugging becomes harder

### Failure Scenario / Pitfall

Race condition due to shared memory:

```python
counter = 0

def increment():
    global counter
    counter += 1
```

Multiple threads may modify `counter` simultaneously.

### Common Problems / Fixes

| Problem                | Fix                        |
| ---------------------- | -------------------------- |
| Race condition         | Use locks                  |
| Deadlock               | Maintain lock ordering     |
| Shared data corruption | Use thread-safe structures |
| Blocking operations    | Use thread pools           |

### Example

```python
import threading
import time

def task(name):
    print(f"{name} started")
    time.sleep(2)
    print(f"{name} completed")

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

### Interview Answer

> Multithreading runs multiple threads inside a process to achieve concurrency. It works best for I/O-bound tasks because threads can execute while others wait, but Python's GIL limits its effectiveness for CPU-intensive workloads.

---

## 10) Multiprocessing

### Definition

Multiprocessing uses **multiple independent processes** to execute tasks in parallel.

Structure:

```text
OS
 ├── Process1
 ├── Process2
 └── Process3
```

Each process:

* Has separate memory
* Runs independently
* Can use different CPU cores

### Use When

* Image processing
* Machine learning
* Scientific calculations
* Data processing
* CPU-heavy tasks

### Avoid When

* Lightweight tasks
* Frequent inter-process communication is required

### Advantages / Tradeoffs

**Advantages**

* True parallel execution
* Uses multiple CPU cores
* Avoids GIL limitations

**Tradeoffs**

* Higher memory usage
* Process creation overhead

### Limitations

* Expensive process startup
* Data sharing is harder

### Failure Scenario / Pitfall

Creating excessive processes:

```python
for i in range(1000):
    Process(target=task)
```

Too many processes increase overhead and hurt performance.

### Common Problems / Fixes

| Problem                | Fix                 |
| ---------------------- | ------------------- |
| High memory use        | Use process pools   |
| Communication overhead | Use queues          |
| Too many workers       | Limit process count |

### Example

```python
from multiprocessing import Process

def task():
    print("Processing")

p1 = Process(target=task)
p2 = Process(target=task)

p1.start()
p2.start()

p1.join()
p2.join()
```

### Interview Answer

> Multiprocessing executes tasks using multiple processes with separate memory spaces. It provides true parallelism and is preferred for CPU-bound workloads.

---

## 11) Python GIL

### Definition

GIL (Global Interpreter Lock) is a mechanism in CPython that allows **only one thread to execute Python bytecode at a time**.

Think:

```text
Thread1 → Running Python code

Thread2 → Waiting

Thread3 → Waiting
```

Even with many threads:

```text
Only one thread executes Python bytecode simultaneously
```

### Use When

You don't "use" GIL directly. You consider it when choosing:

* Multithreading
* Multiprocessing
* AsyncIO

### Avoid When

Do not depend on threads for:

* Heavy calculations
* CPU-intensive processing

### Advantages / Tradeoffs

**Advantages**

* Simpler memory management
* Prevents many memory corruption issues

**Tradeoffs**

* Reduces thread performance for CPU-heavy tasks

### Limitations

* Prevents true thread parallelism in CPython

### Failure Scenario / Pitfall

Developer assumes threads use all cores:

```python
import threading

def calculate():
    for i in range(10**8):
        pass

t1=threading.Thread(target=calculate)
t2=threading.Thread(target=calculate)

t1.start()
t2.start()
```

Performance may barely improve.

### Common Problems / Fixes

| Problem                   | Fix                      |
| ------------------------- | ------------------------ |
| Slow CPU work             | Use multiprocessing      |
| Poor scaling              | Use process pools        |
| Wrong architecture choice | Identify CPU vs I/O work |

### Example

```python
from multiprocessing import Pool

def square(x):
    return x*x

with Pool(4) as p:
    print(p.map(square,[1,2,3,4]))
```

### Interview Answer

> Python's GIL is a lock in CPython that allows only one thread to execute Python bytecode at a time. It simplifies memory management but limits multithreading performance for CPU-bound tasks.

---

## 12) Race Condition

### Definition

A race condition occurs when **multiple threads or processes access shared data simultaneously and the result depends on execution timing.**

Example:

```text
Thread1 reads counter = 5
Thread2 reads counter = 5

Thread1 writes 6
Thread2 writes 6
```

Expected:

```text
7
```

Actual:

```text
6
```

### Use When

Never intentionally.

Race conditions are bugs.

### Avoid When

* Shared variables
* Shared resources
* Concurrent updates

### Advantages / Tradeoffs

**Advantages**

* None

**Tradeoffs**

* Data corruption
* Unpredictable behavior

### Limitations

* Difficult to reproduce
* Hard to debug

### Failure Scenario / Pitfall

```python
import threading

counter=0

def increment():
    global counter

    for _ in range(100000):
        counter += 1

threads=[]

for _ in range(5):
    t=threading.Thread(target=increment)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(counter)
```

Expected:

```text
500000
```

Actual:

```text
Unpredictable
```

### Common Problems / Fixes

| Problem                    | Fix    |
| -------------------------- | ------ |
| Shared variable corruption | Lock   |
| Simultaneous updates       | Mutex  |
| Complex synchronization    | Queues |

### Example (Fix)

```python
import threading

counter=0
lock=threading.Lock()

def increment():
    global counter

    for _ in range(100000):

        with lock:
            counter += 1
```

### Interview Answer

> A race condition occurs when multiple execution units access shared data concurrently and outcomes depend on timing. It can lead to inconsistent results and is typically prevented using locks, mutexes, or synchronization mechanisms.

Next batch: **Deadlock, AsyncIO, Decorators, Generators**.

## 13) Deadlock

### Definition

A deadlock occurs when **two or more threads/processes wait forever for each other to release resources**.

Think:

```text
Thread1 → holds LockA → waiting for LockB

Thread2 → holds LockB → waiting for LockA
```

Nobody proceeds.

### Use When

Never intentionally.

Deadlocks are problems, not techniques.

### Avoid When

* Multiple locks are used
* Nested locking exists
* Circular dependencies exist

### Advantages / Tradeoffs

**Advantages**

* None

**Tradeoffs**

* Application freeze
* Resource wastage
* Difficult debugging

### Limitations

* Hard to reproduce
* Can appear only under specific timing conditions

### Failure Scenario / Pitfall

```python
import threading

lock1 = threading.Lock()
lock2 = threading.Lock()

def task1():
    with lock1:
        with lock2:
            print("Task1")

def task2():
    with lock2:
        with lock1:
            print("Task2")

t1 = threading.Thread(target=task1)
t2 = threading.Thread(target=task2)

t1.start()
t2.start()
```

Possible issue:

```text
Thread1 gets lock1
Thread2 gets lock2

Thread1 waits for lock2
Thread2 waits for lock1

Deadlock
```

### Common Problems / Fixes

| Problem               | Fix                 |
| --------------------- | ------------------- |
| Circular locking      | Maintain lock order |
| Infinite waiting      | Use lock timeout    |
| Multiple nested locks | Minimize lock scope |

### Example (Fix)

```python
import threading

lock1 = threading.Lock()
lock2 = threading.Lock()

def task():

    with lock1:
        with lock2:
            print("Safe execution")
```

Both threads use the same lock order.

### Interview Answer

> A deadlock happens when two or more threads/processes wait indefinitely for resources held by each other. Common prevention methods include consistent lock ordering, minimizing nested locks, and using timeouts.

---

## 14) AsyncIO

### Definition

AsyncIO is Python's framework for **single-threaded asynchronous programming using an event loop**.

It allows tasks to pause while waiting and lets other tasks run.

Flow:

```text
Event Loop
     ↓
Task1 waiting → Task2 runs
Task2 waiting → Task3 runs
```

### Use When

* API requests
* Database calls
* Web servers
* Microservices
* Real-time systems

### Avoid When

* Heavy CPU calculations
* Long blocking functions
* Small scripts

### Advantages / Tradeoffs

**Advantages**

* High throughput
* Low memory usage
* Handles many concurrent requests

**Tradeoffs**

* More complex debugging
* Requires async-compatible libraries

### Limitations

* Not true parallelism
* Blocking code freezes event loop

### Failure Scenario / Pitfall

Wrong:

```python
import asyncio
import time

async def task():
    time.sleep(5)
```

`time.sleep()` blocks the event loop.

### Common Problems / Fixes

| Problem        | Fix                         |
| -------------- | --------------------------- |
| Blocking code  | Use `await asyncio.sleep()` |
| Missing await  | Add `await`                 |
| Sync libraries | Use async libraries         |

### Example

```python
import asyncio

async def task(name, seconds):

    print(f"{name} started")

    await asyncio.sleep(seconds)

    print(f"{name} completed")

async def main():

    await asyncio.gather(
        task("Task1",3),
        task("Task2",2)
    )

asyncio.run(main())
```

### Interview Answer

> AsyncIO is Python's event-driven asynchronous framework that allows multiple I/O tasks to progress concurrently using an event loop. It is highly efficient for I/O-bound applications.

---

## 15) Decorators

### Definition

A decorator is a function that **modifies or extends another function's behavior without changing its source code**.

Syntax:

```python
@decorator_name
```

### Use When

* Logging
* Authentication
* Validation
* Caching
* Timing execution

### Avoid When

* Logic becomes difficult to understand
* Too many nested decorators exist

### Advantages / Tradeoffs

**Advantages**

* Reusable logic
* Cleaner code
* Less duplication

**Tradeoffs**

* Can hide actual execution flow

### Limitations

* Debugging can be harder
* Execution order matters

### Failure Scenario / Pitfall

Without preserving metadata:

```python
def my_decorator(func):

    def wrapper():
        return func()

    return wrapper
```

Function name becomes:

```text
wrapper
```

instead of original name.

### Common Problems / Fixes

| Problem                 | Fix                     |
| ----------------------- | ----------------------- |
| Metadata loss           | Use `functools.wraps()` |
| Wrong argument handling | Use `*args, **kwargs`   |
| Complex nesting         | Keep decorators focused |

### Example

```python
from functools import wraps

def logger(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print("Function started")

        return func(*args, **kwargs)

    return wrapper


@logger
def greet():

    print("Hello")


greet()
```

Output:

```text
Function started
Hello
```

### Interview Answer

> Decorators are functions that wrap other functions to add extra behavior without modifying the original function code. They are commonly used for logging, authentication, caching, and validation.

---

## 16) Generators

### Definition

Generators are functions that **produce values one at a time using `yield` instead of returning everything at once**.

### Use When

* Large datasets
* Streaming data
* File processing
* Memory optimization

### Avoid When

* Data must be accessed randomly
* Multiple iterations over same values are needed

### Advantages / Tradeoffs

**Advantages**

* Low memory usage
* Lazy evaluation
* Faster startup

**Tradeoffs**

* Can only move forward
* Harder debugging

### Limitations

* Cannot directly access arbitrary elements
* Exhaust after use

### Failure Scenario / Pitfall

```python
gen = (x for x in range(3))

print(list(gen))
print(list(gen))
```

Output:

```text
[0,1,2]
[]
```

Generator is exhausted.

### Common Problems / Fixes

| Problem             | Fix                |
| ------------------- | ------------------ |
| Generator exhausted | Recreate generator |
| Need random access  | Convert to list    |
| Multiple iterations | Store results      |

### Example

```python
def numbers():

    for i in range(5):
        yield i


gen = numbers()

for num in gen:
    print(num)
```

### Interview Answer

> Generators produce values lazily using `yield`, allowing efficient memory usage by generating values on demand instead of storing everything in memory.

Next batch: **Pydantic, FastAPI Dependency Injection, Middleware, PostgreSQL Optimization**.

## 17) Pydantic

### Definition

[Pydantic Official Documentation](https://docs.pydantic.dev/?utm_source=chatgpt.com)

Pydantic is a Python library for **data validation, parsing, and serialization using Python type hints**.

It automatically:

* Validates input data
* Converts types when possible
* Generates structured models
* Produces clear validation errors

Example flow:

```text
Request JSON
      ↓
Pydantic Model
      ↓
Validated Python Object
```

### Use When

* FastAPI request/response models
* API validation
* Configuration management
* Structured data processing

### Avoid When

* Very simple scripts
* Performance-critical code with massive object creation
* Dynamic schemas that constantly change

### Advantages / Tradeoffs

**Advantages**

* Automatic validation
* Cleaner code
* Better IDE support
* Strong typing

**Tradeoffs**

* Validation adds overhead
* Learning model configuration takes time

### Limitations

* Complex nested models become difficult
* Validation costs CPU time

### Failure Scenario / Pitfall

Wrong input:

```python
from pydantic import BaseModel

class User(BaseModel):
    age:int

data=User(age="abc")
```

Error:

```text
ValidationError
```

### Common Problems / Fixes

| Problem                  | Fix                  |
| ------------------------ | -------------------- |
| Validation errors        | Match expected types |
| Optional field confusion | Use `Optional`       |
| Complex nesting          | Split models         |

### Example

```python
from pydantic import BaseModel

class User(BaseModel):
    name:str
    age:int


user=User(
    name="John",
    age="25"
)

print(user)
```

Output:

```text
name='John' age=25
```

Pydantic converts `"25"` → `25`

### Interview Answer

> Pydantic is a data validation library that uses Python type hints to validate, parse, and serialize data automatically. It is heavily used in FastAPI for request and response validation.

---

## 18) FastAPI Dependency Injection

### Definition

Dependency Injection (DI) in FastAPI provides **required resources/functions automatically to endpoints without manually creating them inside each route**.

Typical dependencies:

* Database sessions
* Authentication
* Configuration
* Redis connections
* Services

Flow:

```text
Request
   ↓
Dependency
   ↓
Route Function
```

### Use When

* Database connections
* Authentication
* Shared business logic
* Config objects
* Reusable services

### Avoid When

* Small one-off logic
* Very simple applications

### Advantages / Tradeoffs

**Advantages**

* Reusable code
* Cleaner architecture
* Easier testing
* Loose coupling

**Tradeoffs**

* Can become difficult if overused

### Limitations

* Deep dependency chains become confusing

### Failure Scenario / Pitfall

Creating DB session manually:

```python
@app.get("/")
def users():

    db=Session()
```

Possible issue:

```text
Connection leaks
```

### Common Problems / Fixes

| Problem               | Fix                        |
| --------------------- | -------------------------- |
| Resource leaks        | Use `yield` dependencies   |
| Circular dependencies | Split services             |
| Repeated code         | Create shared dependencies |

### Example

```python
from fastapi import FastAPI, Depends

app=FastAPI()

def get_db():
    return "Database Connection"


@app.get("/users")
def get_users(
    db=Depends(get_db)
):
    return {"db":db}
```

### Interview Answer

> FastAPI dependency injection automatically provides reusable components like database sessions, authentication, or services to route handlers, improving maintainability and testability.

---

## 19) Middleware

### Definition

Middleware is code that executes **before and/or after request processing**.

Request flow:

```text
Request
   ↓
Middleware
   ↓
Route Handler
   ↓
Middleware
   ↓
Response
```

### Use When

* Logging
* Authentication
* Request timing
* CORS
* Metrics collection

### Avoid When

* Business logic
* Database queries specific to one endpoint

### Advantages / Tradeoffs

**Advantages**

* Centralized functionality
* Less duplicate code
* Easy monitoring

**Tradeoffs**

* Too much middleware slows requests

### Limitations

* Debugging middleware chains can be difficult

### Failure Scenario / Pitfall

Heavy DB queries inside middleware:

```python
@app.middleware("http")
async def custom(request,call_next):

    expensive_db_call()

    return await call_next(request)
```

Every request becomes slow.

### Common Problems / Fixes

| Problem                          | Fix                     |
| -------------------------------- | ----------------------- |
| Slow middleware                  | Keep logic lightweight  |
| Unexpected request modifications | Validate changes        |
| Too many middleware layers       | Reduce chain complexity |

### Example

```python
from fastapi import FastAPI
import time

app=FastAPI()

@app.middleware("http")
async def log_time(
    request,
    call_next
):

    start=time.time()

    response=await call_next(request)

    print(
        time.time()-start
    )

    return response
```

### Interview Answer

> Middleware intercepts requests and responses to perform cross-cutting concerns such as logging, authentication, timing, and monitoring.

---

## 20) PostgreSQL Optimization

### Definition

PostgreSQL optimization means **improving database performance for faster queries and better resource usage**.

Areas:

* Query optimization
* Indexing
* Connection handling
* Schema design

### Use When

* Slow queries
* Large datasets
* High traffic applications
* Performance bottlenecks

### Avoid When

* Small applications without performance issues
* Premature optimization

### Advantages / Tradeoffs

**Advantages**

* Faster responses
* Better scalability
* Lower resource usage

**Tradeoffs**

* Additional complexity
* Index maintenance cost

### Limitations

* More indexes increase storage
* Wrong optimization can reduce performance

### Failure Scenario / Pitfall

Missing index:

```sql
SELECT *
FROM users
WHERE email='abc@test.com';
```

With millions of rows:

```text
Full table scan
```

### Common Problems / Fixes

| Problem                 | Fix                    |
| ----------------------- | ---------------------- |
| Slow search             | Add indexes            |
| Too many DB connections | Use connection pooling |
| N+1 queries             | Use JOIN/prefetch      |
| Slow query plans        | Analyze using EXPLAIN  |

### Example

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Query analysis:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email='abc@test.com';
```

### Interview Answer

> PostgreSQL optimization involves improving query execution and resource utilization through indexing, query tuning, connection pooling, and efficient schema design.

Next batch: **Redis, Docker, Logging**.


## 21) Redis

[Redis Official Website](https://redis.io/?utm_source=chatgpt.com)

### Definition

Redis is an **in-memory key-value data store** used for:

* Caching
* Session storage
* Pub/Sub messaging
* Rate limiting
* Queues
* Real-time analytics

Think:

```text
Application
      ↓
Redis (RAM)
      ↓
Database
```

Because data is stored in RAM, Redis is extremely fast.

### Use When

* Frequently accessed data
* API response caching
* User sessions
* Rate limiting
* Real-time applications
* Message queues

### Avoid When

* Data size exceeds available memory
* Complex relational queries are needed
* Long-term storage is the primary requirement

### Advantages / Tradeoffs

**Advantages**

* Very fast (memory-based)
* Supports multiple data structures
* Reduces database load
* Supports Pub/Sub

**Tradeoffs**

* RAM is expensive
* Data persistence may require configuration
* Large datasets increase memory usage

### Limitations

* Not a replacement for relational databases
* Limited query capabilities compared to PostgreSQL

### Failure Scenario / Pitfall

Caching without expiration:

```python
redis.set("user:1", data)
```

Issue:

```text
Memory usage continuously grows
```

### Common Problems / Fixes

| Problem           | Fix                  |
| ----------------- | -------------------- |
| Memory growth     | Use TTL (`expire`)   |
| Cache stampede    | Use cache locking    |
| Stale data        | Invalidate cache     |
| Connection issues | Use connection pools |

### Example

```python
import redis

r = redis.Redis(
    host='localhost',
    port=6379,
    decode_responses=True
)

r.set("name","John", ex=60)

print(r.get("name"))
```

### Interview Answer

> Redis is an in-memory key-value data store commonly used for caching, session management, rate limiting, and messaging. It improves performance by reducing database access and providing extremely fast data retrieval.

---

## 22) Docker

[Docker Official Website](https://www.docker.com/?utm_source=chatgpt.com)

### Definition

Docker is a platform for **packaging applications with all dependencies into containers**, ensuring the application runs consistently across environments.

Think:

```text
Application
+ Python
+ Dependencies
+ OS libraries
----------------
Container
```

### Use When

* Microservices
* Deployment consistency
* CI/CD pipelines
* Local development environments
* Scalable applications

### Avoid When

* Very small scripts
* Applications requiring full virtual machines
* Extremely hardware-specific workloads

### Advantages / Tradeoffs

**Advantages**

* "Works on my machine" problem disappears
* Easy deployment
* Isolation between applications
* Portable environments

**Tradeoffs**

* Learning curve
* Additional resource overhead
* Networking complexity

### Limitations

* Containers share host kernel
* Persistent storage needs configuration

### Failure Scenario / Pitfall

Large Docker image:

```dockerfile
FROM python:latest
COPY . .
RUN pip install -r requirements.txt
```

Issue:

```text
Huge image size
Slow deployments
```

### Common Problems / Fixes

| Problem                     | Fix                       |
| --------------------------- | ------------------------- |
| Large image                 | Use slim/alpine images    |
| Container exits immediately | Keep main process running |
| Lost data                   | Use volumes               |
| Slow builds                 | Use layer caching         |

### Example

**Dockerfile**

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python","app.py"]
```

Build:

```bash
docker build -t myapp .
```

Run:

```bash
docker run -p 8000:8000 myapp
```

### Interview Answer

> Docker is a containerization platform that packages applications and their dependencies into isolated containers, ensuring consistency across development, testing, and production environments.

---

## 23) Logging

### Definition

Logging is the process of **recording application events, errors, warnings, and runtime information** for monitoring and debugging.

Log levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

### Use When

* Production applications
* Monitoring
* Debugging
* Auditing
* Error tracking

### Avoid When

* Logging sensitive data
* Excessive debug logs in production

### Advantages / Tradeoffs

**Advantages**

* Easier debugging
* Better monitoring
* Helps diagnose failures
* Improves observability

**Tradeoffs**

* Excessive logging affects performance
* Large logs consume storage

### Limitations

* Poor log design creates noise
* Logs alone may not reveal root causes

### Failure Scenario / Pitfall

Bad:

```python
logger.info(
    f"Password={password}"
)
```

Issue:

```text
Sensitive information exposed
```

### Common Problems / Fixes

| Problem                 | Fix                 |
| ----------------------- | ------------------- |
| Log files too large     | Use log rotation    |
| Missing context         | Add request IDs     |
| Sensitive data exposure | Mask data           |
| Hard debugging          | Use structured logs |

### Example

```python
import logging

logging.basicConfig(
    level=logging.INFO
)

logger = logging.getLogger(__name__)

logger.info(
    "Application started"
)

try:
    x=10/0
except Exception as e:
    logger.error(
        f"Error: {e}"
    )
```

Output:

```text
INFO: Application started
ERROR: division by zero
```

### Interview Answer

> Logging is the process of recording application events and errors to help monitor, debug, and troubleshoot systems. Effective logging uses proper log levels, context information, and avoids sensitive data exposure.

You now have all 23 topics covered in the same structure. A useful next step would be creating a **single interview cheat sheet** showing relationships like:

```text
CPU-bound → Multiprocessing
I/O-bound → AsyncIO / Multithreading

Concurrency ≠ Parallelism

Thread → Shared memory
Process → Separate memory

Decorator → Add behavior
Generator → Lazy evaluation
```

That condensed sheet is often enough for rapid interview revision.

# 1) Background Tasks

## Definition

Background tasks are operations executed **separately from the main request-response cycle**, allowing users to get a response immediately while long-running work continues.

Flow:

```text
Client Request
      ↓
API Response Returned
      ↓
Background Task Executes
```

Examples:

* Send emails
* Generate reports
* Process images
* Notifications
* Audit logs

---

## Use When

* Email sending
* Logging
* File processing
* Notifications
* Non-critical tasks

---

## Avoid When

* Heavy CPU work
* Long-running tasks
* Critical business workflows
* Distributed systems requiring reliability

---

## Advantages / Tradeoffs

**Advantages**

* Faster API response
* Better user experience
* Less request blocking

**Tradeoffs**

* Harder monitoring
* Failure handling complexity

---

## Limitations

* Task may fail silently
* Process crash can lose tasks
* No retry mechanism by default

---

## Failure scenario / Pitfall

```text
API returns success
↓
Server crashes
↓
Background task never executes
```

---

## Common Problems / Fixes

| Problem         | Fix                   |
| --------------- | --------------------- |
| Lost tasks      | Use Celery            |
| Long execution  | Move to worker system |
| Silent failures | Add logging           |

---

## Example

```python
def send_email():
    print("Sending Email")

@app.post("/users")
def create_user():

    background_task(send_email)

    return {
        "message":"User created"
    }
```

---

## Interview Answer

> Background tasks execute work separately from the request-response cycle to improve API responsiveness. They are suitable for lightweight asynchronous operations like sending emails or logging.

---

# 2) Redis (for Background Processing)

[Redis Official Website](https://redis.io?utm_source=chatgpt.com)

## Definition

Redis often acts as a **temporary storage or message broker** between applications and worker systems.

Flow:

```text
Application
     ↓
Redis Queue
     ↓
Worker
```

Redis itself does not execute tasks.

Redis stores:

* Jobs
* Messages
* Task states
* Results

---

## Use When

* Queue storage
* Celery broker
* Task state tracking
* Caching

---

## Avoid When

* Permanent storage
* Complex relational data

---

## Advantages / Tradeoffs

**Advantages**

* Very fast
* Lightweight
* Easy integration

**Tradeoffs**

* Memory-based
* Large datasets increase cost

---

## Limitations

* Requires persistence configuration
* Can lose data if misconfigured

---

## Failure scenario / Pitfall

```text
Redis memory full
↓
New tasks rejected
```

---

## Common Problems / Fixes

| Problem              | Fix                |
| -------------------- | ------------------ |
| Memory growth        | TTL                |
| Lost messages        | Persistence        |
| Too many connections | Connection pooling |

---

## Example

```python
redis.lpush(
    "jobs",
    "send_email"
)
```

---

## Interview Answer

> Redis commonly acts as a broker for background processing systems by temporarily storing jobs and passing them to workers.

---

# 3) Celery

[Celery Official Documentation](https://docs.celeryq.dev?utm_source=chatgpt.com)

## Definition

Celery is a **distributed task queue system** used for executing background jobs asynchronously.

Flow:

```text
FastAPI
    ↓
Redis/RabbitMQ
    ↓
Celery Worker
    ↓
Task Execution
```

---

## Use When

* Email systems
* Report generation
* Video processing
* Scheduled tasks
* Heavy background jobs

---

## Avoid When

* Small projects
* Simple lightweight tasks
* Quick operations

---

## Advantages / Tradeoffs

**Advantages**

* Retry support
* Scheduling
* Distributed workers
* Monitoring

**Tradeoffs**

* More infrastructure
* Setup complexity

---

## Limitations

* Additional services needed
* Debugging can be harder

---

## Failure scenario / Pitfall

```text
Task added
↓
Worker not running
↓
Task remains pending
```

---

## Common Problems / Fixes

| Problem             | Fix                  |
| ------------------- | -------------------- |
| Pending tasks       | Start workers        |
| Duplicate execution | Use idempotent tasks |
| Task failures       | Retries              |

---

## Example

**celery_app.py**

```python
from celery import Celery

app = Celery(
    "tasks",
    broker="redis://localhost:6379"
)

@app.task
def send_email():

    print("Email sent")
```

Trigger task:

```python
send_email.delay()
```

Worker:

```bash
celery -A celery_app worker --loglevel=info
```

---

## Interview Answer

> Celery is a distributed task queue system that executes background jobs asynchronously using brokers like Redis or RabbitMQ.

---

# 4) FastAPI Background Tasks

[FastAPI Background Tasks Documentation](https://fastapi.tiangolo.com/tutorial/background-tasks/?utm_source=chatgpt.com)

## Definition

FastAPI provides a built-in `BackgroundTasks` class to execute tasks **after returning a response**.

Flow:

```text
Request
   ↓
Return response immediately
   ↓
BackgroundTasks executes
```

---

## Use When

* Email notifications
* Logging
* Small file operations
* Lightweight async work

---

## Avoid When

* CPU-intensive work
* Long-running jobs
* Critical business tasks

---

## Advantages / Tradeoffs

**Advantages**

* Easy implementation
* No external service
* Built into FastAPI

**Tradeoffs**

* Runs in same application process

---

## Limitations

* No retries
* No scheduling
* No worker scaling
* Lost on app crash

---

## Failure scenario / Pitfall

```text
Request completed
↓
Server restarts
↓
Task disappears
```

---

## Common Problems / Fixes

| Problem               | Fix         |
| --------------------- | ----------- |
| Lost tasks            | Use Celery  |
| Heavy tasks block app | Use workers |
| Silent errors         | Logging     |

---

## Example

```python
from fastapi import FastAPI
from fastapi import BackgroundTasks

app=FastAPI()

def send_email(email):

    print(
        f"Email sent to {email}"
    )

@app.post("/register")
async def register(
    background_tasks: BackgroundTasks
):

    background_tasks.add_task(
        send_email,
        "abc@test.com"
    )

    return {
        "message":"Registered"
    }
```

---

## Interview Answer

> FastAPI BackgroundTasks execute lightweight operations after sending a response. They are suitable for simple tasks like emails or logging but not for long-running distributed workloads.

---

# Quick Interview Comparison

| Feature         | Background Task | FastAPI BackgroundTasks | Celery |
| --------------- | --------------: | ----------------------: | -----: |
| Separate worker |              No |                      No |    Yes |
| Retry support   |              No |                      No |    Yes |
| Scheduling      |              No |                      No |    Yes |
| Distributed     |              No |                      No |    Yes |
| Heavy tasks     |              No |                      No |    Yes |
| Reliability     |             Low |                     Low |   High |
| Complexity      |             Low |                     Low | Medium |

### Common interview question

**Why use Celery instead of FastAPI BackgroundTasks?**

**Answer:**

> FastAPI BackgroundTasks run in the same process and are suitable for lightweight work. Celery provides distributed workers, retries, monitoring, scheduling, and reliability for long-running production tasks.

