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
