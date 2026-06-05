# Redis — Full Notes (Part 1: Foundations, Types, Caching, Rate Limiting)

[Redis Documentation](https://redis.io/docs/?utm_source=chatgpt.com)

---

# 1. What is Redis?

Redis (**REmote DIctionary Server**) is an **in-memory data structure store** used as:

* Cache
* Database
* Message broker
* Queue
* Rate limiter
* Session store
* Pub/Sub system
* Real-time analytics engine

Redis primarily stores data in **RAM**, making reads/writes extremely fast.

Typical architecture:

```text
Client
   ↓
Redis (RAM)
   ↓
PostgreSQL/MySQL
```

---

# 2. Redis Data Types ("Types of Redis")

People often say "types of Redis", but usually they mean **Redis data structures**.

## A. String

Most common data type.

Stores:

* Text
* Numbers
* JSON
* Serialized objects

Example:

```python
r.set("username","kk")

print(
    r.get("username")
)
```

Output:

```text
kk
```

Use cases:

* Cache values
* Session IDs
* Counters
* Tokens

Commands:

```bash
SET key value
GET key
INCR key
DECR key
```

---

## B. List

Ordered collection.

```python
r.lpush("tasks","Task1")
r.lpush("tasks","Task2")

r.lrange("tasks",0,-1)
```

Output:

```text
Task2
Task1
```

Use cases:

* Queues
* Task processing
* Chat messages

Commands:

```bash
LPUSH
RPUSH
LPOP
RPOP
LRANGE
```

---

## C. Set

Unordered collection of unique values.

```python
r.sadd(
    "skills",
    "Python"
)

r.sadd(
    "skills",
    "Redis"
)
```

Use cases:

* Unique visitors
* Tags
* Followers

Commands:

```bash
SADD
SMEMBERS
SREM
```

---

## D. Sorted Set (ZSET)

Stores values with scores.

```python
r.zadd(
    "leaderboard",
    {"John":100}
)
```

Use cases:

* Leaderboards
* Ranking systems
* Priority queues

Commands:

```bash
ZADD
ZRANGE
ZREM
```

---

## E. Hash

Stores objects.

```python
r.hset(
    "user:1",
    mapping={
        "name":"KK",
        "age":25
    }
)
```

Use cases:

* User profiles
* Product details
* Config storage

Commands:

```bash
HSET
HGET
HGETALL
```

---

## F. Stream

For event processing.

```python
r.xadd(
    "orders",
    {
        "product":"Laptop"
    }
)
```

Use cases:

* Event systems
* Kafka-like workflows
* Logs

Commands:

```bash
XADD
XRANGE
XREAD
```

---

## G. Bitmap

Stores binary data efficiently.

Use cases:

* User online status
* Feature flags

---

## H. HyperLogLog

Estimates unique counts with low memory.

Use cases:

* Website visitors

Example:

```bash
PFADD visitors user1
PFCOUNT visitors
```

---

# 3. Redis as Cache

Caching is Redis's most common use case.

Flow:

```text
Request
   ↓
Check Redis

Found?
 ├─ Yes → Return
 │
 └─ No
      ↓
Database
      ↓
Save to Redis
      ↓
Return response
```

---

# Cache Types

---

## A. Cache-Aside Pattern (Lazy Loading)

Most common approach.

Flow:

```text
Request
   ↓
Redis

Miss?
   ↓
Database
   ↓
Save in Redis
```

Example:

```python
user=r.get("user:1")

if not user:

    user=db.fetch()

    r.set(
        "user:1",
        user,
        ex=300
    )
```

Use when:

* Read-heavy systems

Advantages:

* Simple
* Saves memory

Problems:

* First request slow
* Cache miss latency

---

## B. Write-Through Cache

Data written to:

```text
Application
   ↓
Redis
   ↓
Database
```

Example:

```python
r.set(
    "user:1",
    data
)

db.save(data)
```

Advantages:

* Cache always updated

Disadvantages:

* Slower writes

---

## C. Write-Behind Cache

```text
Application
    ↓
Redis
    ↓
Database later
```

Advantages:

* Fast writes

Disadvantages:

* Risk of data loss

Use:

* Analytics
* Logs

---

## D. Read-Through Cache

Application:

```text
Request
    ↓
Redis automatically loads from DB
```

Application doesn't manage cache logic.

Advantages:

* Cleaner architecture

Disadvantages:

* More setup complexity

---

# 4. Cache Eviction Policies

Redis memory is limited.

When memory is full:

Redis removes data according to policy.

---

## noeviction

```text
Memory full → Error
```

---

## allkeys-lru

Remove:

```text
Least Recently Used
```

Most common production setting.

---

## volatile-lru

Remove:

```text
Least recently used among keys with TTL
```

---

## allkeys-random

Remove random keys.

---

## volatile-ttl

Remove:

```text
Keys closest to expiration
```

---

# 5. Redis TTL (Time To Live)

TTL automatically removes data.

Example:

```python
r.set(
    "token",
    "abc123",
    ex=60
)
```

Expires:

```text
after 60 seconds
```

Check:

```python
r.ttl("token")
```

---

# 6. Redis as Rate Limiter

Rate limiting:

```text
Limit requests per user/IP
```

Examples:

* Login attempts
* API usage
* OTP requests

---

## Fixed Window Algorithm

Example:

```text
Limit:
100 requests/minute
```

Flow:

```python
count=r.incr(
    "user123"
)

if count==1:
    r.expire(
        "user123",
        60
    )

if count>100:
    return "Blocked"
```

Problem:

```text
Burst traffic near boundary
```

Example:

```text
11:59 → 100 requests
12:00 → 100 requests

Total=200
```

---

## Sliding Window Algorithm

Tracks requests continuously.

Flow:

```text
Current Time
      ↓
Remove expired requests
      ↓
Count requests
```

Advantages:

* More accurate

Disadvantages:

* More memory usage

Usually implemented with:

```text
Sorted Sets (ZSET)
```

---

## Token Bucket Algorithm

Flow:

```text
Bucket: 10 tokens

Request:
Remove token

No token:
Reject request
```

Advantages:

* Allows bursts

Use cases:

* APIs

---

## Leaky Bucket Algorithm

Flow:

```text
Requests enter bucket

Output rate fixed
```

Advantages:

* Smooth traffic flow

---

# Common Redis Production Problems

| Problem             |                     Cause | Fix               |
| ------------------- | ------------------------: | ----------------- |
| Memory explosion    |                    No TTL | Add expiration    |
| Cache stampede      |       Simultaneous misses | Use locks         |
| Cache penetration   |   Invalid requests hit DB | Cache null values |
| Cache avalanche     | Many keys expire together | Randomize TTL     |
| Slow performance    |                Large keys | Split data        |
| Connection overload |          Too many clients | Connection pool   |

---

# Interview Questions

### Why Redis instead of database caching?

**Answer:**

> Redis stores data in memory and provides extremely fast reads/writes compared to disk-based databases.

---

### Which cache strategy is most common?

**Answer:**

> Cache-aside is most commonly used because it is simple and efficient.

---

### Why use Sorted Sets in rate limiting?

**Answer:**

> Sorted Sets store timestamps as scores, making it easy to remove expired requests and implement sliding windows.

---

### Why add TTL?

**Answer:**

> TTL prevents stale data and avoids memory growth.

---

Next useful section would be **Redis Pub/Sub, Redis Streams, Redis Distributed Locks, Redis Cluster, Redis Sentinel, Session Storage, Celery + Redis integration, and Redis in FastAPI architecture.**

# Redis — Full Notes (Part 2: Pub/Sub, Streams, Distributed Locks, Cluster, Sentinel, Sessions, Celery, FastAPI)

[Redis Documentation](https://redis.io/docs/?utm_source=chatgpt.com)

---

# 7. Redis Pub/Sub

### Definition

Pub/Sub (**Publisher/Subscriber**) is a messaging pattern where:

* Publisher sends messages
* Subscribers receive messages
* Publisher and subscribers do not know each other

Flow:

```text
Publisher
     ↓
   Channel
   ↙      ↘
Sub1      Sub2
```

---

### Use When

* Real-time notifications
* Chat systems
* Live dashboards
* Event broadcasting

---

### Avoid When

* Messages cannot be lost
* Message persistence is required
* Reliable delivery is mandatory

---

### Advantages / Tradeoffs

**Advantages**

* Very fast
* Simple implementation
* Real-time delivery

**Tradeoffs**

* Messages are temporary
* No delivery guarantee

---

### Limitation

If subscriber disconnects:

```text
Message lost permanently
```

---

### Example

Publisher:

```python
import redis

r = redis.Redis()

r.publish(
    "chat",
    "Hello everyone"
)
```

Subscriber:

```python
import redis

r = redis.Redis()

pubsub = r.pubsub()

pubsub.subscribe("chat")

for message in pubsub.listen():
    print(message)
```

---

### Failure Scenario

```text
Subscriber offline
        ↓
Message sent
        ↓
Message lost
```

---

### Interview Answer

> Redis Pub/Sub is a lightweight messaging system where publishers send messages to channels and subscribers receive them in real time. Messages are not persisted.

---

# 8. Redis Streams

### Definition

Streams are an append-only log structure for event-driven systems.

Flow:

```text
Producer
    ↓
Redis Stream
    ↓
Consumers
```

---

### Difference vs Pub/Sub

| Feature             | Pub/Sub | Streams |
| ------------------- | ------: | ------: |
| Message persistence |      No |     Yes |
| Replay old messages |      No |     Yes |
| Consumer groups     |      No |     Yes |
| Reliability         |     Low |    High |

---

### Use When

* Event processing
* Order systems
* Background jobs
* Analytics pipelines

---

### Example

Add message:

```python
r.xadd(
    "orders",
    {
        "user":"kk",
        "item":"laptop"
    }
)
```

Read messages:

```python
r.xread(
    {"orders":"0"}
)
```

---

### Consumer Groups

Multiple workers can share work.

```text
Stream
  ↓
Consumer Group
  ↙        ↘
Worker1   Worker2
```

Advantages:

* Prevent duplicate processing
* Horizontal scaling

---

### Failure Scenario

Without acknowledging messages:

```text
Consumer crashes
      ↓
Message remains pending
```

Fix:

```bash
XACK
```

---

### Interview Answer

> Redis Streams provide durable event storage with consumer groups and message replay capabilities, making them suitable for reliable event-driven systems.

---

# 9. Redis Distributed Locks

### Definition

Distributed locks prevent multiple services from modifying the same resource simultaneously.

Flow:

```text
Service1
    ↓
 Acquire Lock
    ↓
Redis
    ↓
Service2 blocked
```

---

### Use When

* Prevent duplicate payments
* Prevent duplicate order processing
* Scheduled jobs
* Shared resources

---

### Example

Acquire lock:

```python
lock = r.set(
    "payment_lock",
    "locked",
    nx=True,
    ex=10
)
```

Explanation:

```text
nx=True
Only create if absent

ex=10
Expire after 10 sec
```

---

### Problem without expiration

```text
Service crashes
      ↓
Lock never released
      ↓
System blocked forever
```

---

### Redlock Algorithm

Redis distributed lock algorithm using:

```text
Multiple Redis instances
```

Purpose:

```text
Prevent single-node failure
```

---

### Interview Answer

> Redis distributed locks coordinate access across multiple servers using atomic operations and expiration times.

---

# 10. Redis Cluster

### Definition

Redis Cluster distributes data across multiple Redis nodes.

Flow:

```text
Application
      ↓
Redis Cluster
  ↙    ↓    ↘
Node1 Node2 Node3
```

---

### Why use it?

Single Redis server:

```text
Limited RAM
Limited throughput
```

Cluster:

```text
Horizontal scaling
```

---

### Concepts

## Hash Slots

Redis uses:

```text
16384 slots
```

Example:

```text
user:1 → slot 2300

user:2 → slot 9800
```

Slots determine where data lives.

---

### Advantages

* Scales horizontally
* Better throughput
* Fault tolerance

---

### Tradeoffs

* Setup complexity
* Cross-slot operations can be difficult

---

### Failure Scenario

```text
Node failure
```

Fix:

```text
Replication + failover
```

---

### Interview Answer

> Redis Cluster distributes data among multiple nodes using hash slots to provide scalability and high availability.

---

# 11. Redis Sentinel

### Definition

Sentinel provides:

* Monitoring
* Automatic failover
* Notification
* Service discovery

Flow:

```text
Sentinel
     ↓
Master
   ↓
Replica1
Replica2
```

---

### Example Failure

```text
Master crashes
```

Sentinel:

```text
Detect failure
      ↓
Promote replica
      ↓
New master created
```

---

### Advantages

* Automatic failover
* High availability
* Monitoring

---

### Limitations

* Doesn't scale writes
* Different from cluster

---

### Cluster vs Sentinel

| Feature            | Sentinel | Cluster |
| ------------------ | -------: | ------: |
| High availability  |      Yes |     Yes |
| Horizontal scaling |       No |     Yes |
| Automatic failover |      Yes |     Yes |
| Data sharding      |       No |     Yes |

---

### Interview Answer

> Redis Sentinel monitors Redis servers and automatically promotes replicas during failures to provide high availability.

---

# 12. Redis Session Storage

### Definition

Sessions store user information across requests.

Without Redis:

```text
User
 ↓
Server1
```

Problem:

```text
Server2 has no session
```

---

With Redis:

```text
User
 ↓
Load Balancer
 ↓
Server1 Server2 Server3
      ↓
     Redis
```

---

### Store session

```python
r.set(
    "session:123",
    "user_data",
    ex=3600
)
```

---

### Advantages

* Shared among servers
* Fast lookup
* Session expiration

---

### Failure Scenario

No expiration:

```text
Millions of sessions
      ↓
Memory growth
```

---

### Interview Answer

> Redis stores centralized session data that multiple application servers can access, improving scalability.

---

# 13. Celery + Redis

### Definition

Celery is a task queue system.

Redis often acts as:

* Message broker
* Result backend

Flow:

```text
Application
     ↓
Celery Task
     ↓
Redis Queue
     ↓
Worker
```

---

### Example

Task:

```python
from celery import Celery

app = Celery(
    "tasks",
    broker="redis://localhost:6379"
)

@app.task
def send_email():

    return "Email sent"
```

Call:

```python
send_email.delay()
```

---

### Use Cases

* Email sending
* Video processing
* Background jobs
* Scheduled tasks

---

### Interview Answer

> Redis commonly acts as a Celery message broker where tasks are queued and processed asynchronously by workers.

---

# 14. Redis in FastAPI Architecture

Typical production architecture:

```text
Client
   ↓
FastAPI
   ↓
Redis
   ↓
PostgreSQL
```

Redis handles:

* Cache
* Sessions
* Rate limiting
* Background queues
* Pub/Sub
* Temporary data

---

Example:

```python
from fastapi import FastAPI
import redis

app = FastAPI()

r = redis.Redis(
    host="localhost",
    port=6379
)

@app.get("/users/{id}")

async def get_user(id:int):

    cached = r.get(f"user:{id}")

    if cached:
        return cached

    user = db.get_user(id)

    r.set(
        f"user:{id}",
        user,
        ex=300
    )

    return user
```

---

# Common Redis Interview Questions

### Why Redis instead of Memcached?

> Redis supports multiple data structures, persistence, Pub/Sub, Streams, clustering, and transactions.

### Why use Redis in FastAPI?

> Redis reduces database load through caching and supports sessions, rate limiting, queues, and real-time features.

### Sentinel vs Cluster?

> Sentinel provides high availability while Cluster provides horizontal scaling and high availability.

### Pub/Sub vs Streams?

> Pub/Sub is temporary messaging, while Streams provide persistence and reliable processing.

---

You now have a fairly complete Redis roadmap:

```text
Redis Basics
      ↓
Data Types
      ↓
Caching
      ↓
Rate Limiting
      ↓
Pub/Sub
      ↓
Streams
      ↓
Distributed Locks
      ↓
Cluster/Sentinel
      ↓
Sessions
      ↓
Celery
      ↓
FastAPI Integration
```

A valuable next section is **Redis Transactions (MULTI/EXEC), Pipelines, Lua Scripts, Persistence (RDB vs AOF), and Production Deployment Patterns**.

# Redis — Full Notes (Part 3: Transactions, Pipelines, Lua Scripts, Persistence, Production Patterns)

[Redis Documentation](https://redis.io/docs/latest/develop/interact/transactions/?utm_source=chatgpt.com)

---

# 15. Redis Transactions

## Definition

Redis transactions execute **multiple commands as one atomic sequence**.

Commands:

```text
MULTI
EXEC
DISCARD
WATCH
```

Flow:

```text
MULTI
   ↓
Queue commands
   ↓
EXEC
```

---

## Basic Transaction

```python
import redis

r=redis.Redis()

pipe=r.pipeline()

pipe.set("user:1","KK")
pipe.set("user:2","John")

pipe.execute()
```

Equivalent Redis commands:

```bash
MULTI

SET user:1 KK
SET user:2 John

EXEC
```

---

## Use When

* Updating multiple values together
* Financial operations
* Inventory systems
* Consistency requirements

---

## Avoid When

* Large long-running tasks
* Heavy business logic

---

## Advantages / Tradeoffs

**Advantages**

* Groups operations
* Better consistency

**Tradeoffs**

* Doesn't support rollback like SQL databases

---

## Limitation

Redis transaction:

```text
Not ACID like PostgreSQL
```

Redis executes queued commands sequentially.

If one command fails:

```text
Other commands may still execute
```

---

## Failure Scenario

```bash
MULTI

SET age abc
INCR age

EXEC
```

Result:

```text
SET succeeds

INCR fails
```

No rollback occurs.

---

## WATCH (Optimistic Locking)

WATCH monitors keys.

If data changes:

```text
Transaction aborted
```

Example:

```python
r.watch("balance")

current=int(
    r.get("balance")
)

pipe=r.pipeline()

pipe.multi()

pipe.set(
    "balance",
    current-100
)

pipe.execute()
```

---

## Interview Answer

> Redis transactions group commands using MULTI and EXEC. They ensure sequential execution but do not provide full rollback like relational databases.

---

# 16. Redis Pipelines

## Definition

Pipeline sends multiple commands in **one network request**.

Without pipeline:

```text
App
 ↓
Redis
 ↓
App
 ↓
Redis
```

Many network trips.

With pipeline:

```text
App
      ↓
Batch commands
      ↓
Redis
```

---

## Use When

* Bulk inserts
* Bulk updates
* Large writes
* Performance optimization

---

## Avoid When

* Commands depend on previous results

---

## Advantages / Tradeoffs

**Advantages**

* Fewer network calls
* Faster performance

**Tradeoffs**

* Higher memory usage

---

## Example

Without pipeline:

```python
for i in range(1000):

    r.set(
        f"user:{i}",
        i
    )
```

With pipeline:

```python
pipe=r.pipeline()

for i in range(1000):

    pipe.set(
        f"user:{i}",
        i
    )

pipe.execute()
```

---

## Performance Difference

Example:

```text
Without pipeline:
1000 requests

With pipeline:
1 request
```

---

## Failure Scenario

Huge pipeline:

```python
for i in range(10000000):

    pipe.set(...)
```

Problem:

```text
Large memory usage
```

Fix:

```text
Batch in chunks
```

---

## Interview Answer

> Redis pipelines reduce network overhead by batching multiple commands into a single request.

---

# 17. Redis Lua Scripts

## Definition

Redis can execute **Lua scripts atomically on the server side**.

Command:

```bash
EVAL
```

Flow:

```text
Application
      ↓
Lua Script
      ↓
Redis
```

---

## Why use Lua?

Without Lua:

```python
value=r.get("counter")

value+=1

r.set(
    "counter",
    value
)
```

Problem:

```text
Race condition
```

---

With Lua:

```lua
local count=

redis.call(
'INCR',
KEYS[1]
)

return count
```

Atomic execution.

---

## Example

Python:

```python
script="""
return redis.call(
'INCR',
KEYS[1]
)
"""

r.eval(
script,
1,
"counter"
)
```

---

## Use Cases

* Atomic operations
* Rate limiting
* Distributed locks
* Complex business logic

---

## Advantages / Tradeoffs

**Advantages**

* Atomic
* Fast
* Reduces round trips

**Tradeoffs**

* Harder debugging

---

## Interview Answer

> Lua scripts execute atomically inside Redis and are useful for implementing complex operations without race conditions.

---

# 18. Redis Persistence

Redis stores data primarily in memory.

Without persistence:

```text
Redis restart
      ↓
All data lost
```

Persistence solves this.

Two mechanisms:

```text
RDB
AOF
```

---

# RDB (Redis Database Backup)

## Definition

Creates snapshots periodically.

Flow:

```text
Memory
   ↓
Snapshot
   ↓
disk.rdb
```

Configuration:

```bash
save 900 1
save 300 10
save 60 10000
```

Meaning:

```text
After:

900 sec + 1 change
300 sec + 10 changes
60 sec + 10000 changes
```

---

## Advantages

* Small files
* Fast recovery
* Good for backups

---

## Tradeoffs

Possible data loss:

```text
Crash before next snapshot
```

---

# AOF (Append Only File)

## Definition

Logs every write operation.

Flow:

```text
SET name kk
SET age 25
INCR counter
```

Stored in:

```text
appendonly.aof
```

---

## Advantages

* Better durability

---

## Tradeoffs

* Larger files
* Slower writes

---

## AOF Sync Modes

### always

```text
Write every command immediately
```

Fast recovery:

✅

Performance:

❌

---

### everysec

```text
Sync every second
```

Most common production setting.

---

### no

```text
OS decides sync timing
```

Fast:

✅

Potential loss:

❌

---

# RDB vs AOF

| Feature        |    RDB |     AOF |
| -------------- | -----: | ------: |
| File size      |  Small |   Large |
| Recovery speed |   Fast |  Slower |
| Durability     |  Lower |  Higher |
| Backup         | Better | Average |

---

Production commonly:

```text
RDB + AOF together
```

---

# 19. Production Redis Architecture

Typical architecture:

```text
Client
   ↓
Load Balancer
   ↓
FastAPI Servers
   ↓
Redis Cluster
   ↓
PostgreSQL
```

Redis responsibilities:

```text
Cache
Session
Rate limiting
Queue
Pub/Sub
Distributed lock
Temporary data
```

---

# 20. Redis Production Problems

## Cache Stampede

### Problem

```text
Cache expires
      ↓
1000 requests
      ↓
Database hit simultaneously
```

---

### Fix

Locking:

```python
lock=r.set(
    "lock:user:1",
    1,
    nx=True,
    ex=5
)
```

---

## Cache Penetration

### Problem

```text
Invalid IDs repeatedly hit DB
```

Example:

```text
user:-999999
```

---

### Fix

Cache null values:

```python
r.set(
    "user:-999999",
    "NULL",
    ex=60
)
```

---

## Cache Avalanche

### Problem

```text
Millions of keys expire together
```

---

### Fix

Randomize expiration:

```python
import random

ttl=300+random.randint(
1,
60
)
```

---

## Hot Keys

### Problem

```text
Single key receives
millions of requests
```

---

### Fix

```text
Replication
Sharding
Local cache
```

---

# Real FastAPI + Redis Interview Scenario

**Question:**

*"How would you use Redis in a FastAPI production application?"*

**Answer:**

> I would use Redis for API response caching, centralized session storage, rate limiting, background task queues with Celery, distributed locking for shared resources, and Pub/Sub or Streams for real-time features. For high availability I'd use Sentinel or Cluster and configure TTL policies to avoid memory growth.

---

Redis roadmap (complete view):

```text
Redis Basics
      ↓
Data Types
      ↓
Caching
      ↓
Rate Limiting
      ↓
Pub/Sub
      ↓
Streams
      ↓
Locks
      ↓
Cluster
      ↓
Sentinel
      ↓
Transactions
      ↓
Pipelines
      ↓
Lua Scripts
      ↓
Persistence
      ↓
Production Patterns
```

You now have material that reaches from beginner level into many backend interview topics.

