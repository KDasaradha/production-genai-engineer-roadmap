Good scope. For a backend developer (FastAPI/Python/microservices), I would keep these only and not add obscure Redis patterns.

## Recommended Developer Scope

### Cache

1. Cache Aside (Lazy Loading) ⭐ Most important
2. Write Through Cache
3. Write Back (Write Behind)
4. Refresh Ahead Cache

### Queue

1. List-based Queue
2. Blocking Queue
3. Redis Streams ⭐ Most important

### Rate Limiter

1. Fixed Window
2. Sliding Window ⭐ Most common
3. Token Bucket ⭐ Very common in APIs
4. Leaky Bucket

### Session Store

1. Session ID + Redis Hash ⭐ Most common
2. JWT + Redis Blacklist
3. Distributed Sessions ⭐ Important for microservices

### Pub/Sub

1. Basic Pub/Sub
2. Pattern Pub/Sub
3. Streams-based Pub/Sub ⭐ Production preferred

---

# Cache

# 1. Cache Aside (Lazy Loading)

### Definition

Application first checks Redis. If data doesn't exist, fetch from DB and store in Redis.

### Use When

* Product data
* User profiles
* API responses

### Avoid When

* Frequently changing data

### Advantages / Tradeoffs

**Advantages**

* Reduces DB load
* Simple implementation

**Tradeoffs**

* First request is slow

### Limitations

* Cache miss exists

### Failure scenario / Pitfall

```text
Cache expires
↓
1000 users request same data
↓
Database overload
```

### Common Problems / Fixes

| Problem        | Fix     |
| -------------- | ------- |
| Cache stampede | Locking |
| Stale data     | TTL     |

### Example

```python
data=redis.get("user:1")

if not data:
    data=db.get_user(1)
    redis.set("user:1",data)
```

### Interview Answer

> Cache Aside fetches data from cache first and falls back to database on cache misses.

---

# 2. Write Through Cache

### Definition

Writes go to cache and database simultaneously.

### Use When

* Consistent data required

### Avoid When

* High write systems

### Advantages / Tradeoffs

**Advantages**

* Always fresh cache

**Tradeoffs**

* Slower writes

### Limitations

Extra write overhead

### Failure scenario / Pitfall

Database write fails after cache updated.

### Common Problems / Fixes

| Problem         | Fix         |
| --------------- | ----------- |
| Partial updates | Transaction |

### Example

```python
redis.set(key,data)
db.save(data)
```

### Interview Answer

> Write Through writes data to cache and database together for consistency.

---

# 3. Write Back (Write Behind)

### Definition

Writes happen to cache immediately and DB later.

### Use When

* Heavy write systems

### Avoid When

* Critical financial systems

### Advantages / Tradeoffs

**Advantages**

* Fast writes

**Tradeoffs**

* Risk of data loss

### Limitations

Needs background sync

### Failure scenario / Pitfall

```text
Redis crashes before DB update
```

### Common Problems / Fixes

| Problem   | Fix         |
| --------- | ----------- |
| Data loss | Persistence |

### Example

```python
redis.set(key,data)

# background save to DB
```

### Interview Answer

> Write Back stores data in cache first and updates database asynchronously.

---

# 4. Refresh Ahead Cache

### Definition

Refresh data before cache expires.

### Use When

* High traffic APIs

### Avoid When

* Low traffic systems

### Advantages / Tradeoffs

**Advantages**

* Reduces cache misses

**Tradeoffs**

* Extra refresh cost

### Limitations

Can refresh unused data

### Failure scenario / Pitfall

Unnecessary refreshes waste resources.

### Common Problems / Fixes

| Problem          | Fix                   |
| ---------------- | --------------------- |
| Refresh overhead | Refresh hot keys only |

### Example

```python
background_refresh()
```

### Interview Answer

> Refresh Ahead updates cache before expiration to avoid misses.

---

# Queue

# 1. List-based Queue

### Definition

Uses Redis List as FIFO queue.

### Use When

* Small background jobs

### Avoid When

* Complex message processing

### Advantages / Tradeoffs

**Advantages**

* Easy setup

**Tradeoffs**

* No retry support

### Limitations

No message tracking

### Failure scenario / Pitfall

Consumer crashes after reading message.

### Common Problems / Fixes

| Problem   | Fix         |
| --------- | ----------- |
| Lost jobs | Use Streams |

### Example

```python
redis.lpush("jobs","email")
redis.rpop("jobs")
```

### Interview Answer

> Redis Lists provide simple FIFO queues.

---

# 2. Blocking Queue

### Definition

Consumer waits until message arrives.

### Use When

* Worker systems

### Avoid When

* Complex processing

### Advantages / Tradeoffs

**Advantages**

* No polling

**Tradeoffs**

* Limited features

### Limitations

No acknowledgement

### Failure scenario / Pitfall

Consumer dies during processing.

### Common Problems / Fixes

| Problem   | Fix     |
| --------- | ------- |
| Lost jobs | Streams |

### Example

```python
redis.blpop("jobs")
```

### Interview Answer

> Blocking queues wait for jobs instead of continuously polling.

---

# 3. Redis Streams

### Definition

Advanced queue supporting message tracking.

### Use When

* Microservices
* Event systems

### Avoid When

* Very simple queues

### Advantages / Tradeoffs

**Advantages**

* Retry support
* Consumer groups

**Tradeoffs**

* More complex

### Limitations

Additional memory

### Failure scenario / Pitfall

Pending messages remain unprocessed.

### Common Problems / Fixes

| Problem        | Fix                  |
| -------------- | -------------------- |
| Stuck messages | Acknowledge messages |

### Example

```python
redis.xadd(
    "orders",
    {"id":101}
)
```

### Interview Answer

> Redis Streams are durable queues with consumer groups and acknowledgements.

---

# Rate Limiter

# 1. Fixed Window

### Definition

Request count resets after fixed time.

### Use When

* Simple APIs

### Avoid When

* Traffic spikes

### Advantages / Tradeoffs

**Advantages**

* Simple

**Tradeoffs**

* Burst issue

### Limitations

Boundary problem

### Failure scenario / Pitfall

```text
99 req at 1:59
99 req at 2:00
```

### Common Problems / Fixes

| Problem       | Fix            |
| ------------- | -------------- |
| Burst traffic | Sliding Window |

### Example

```python
INCR user1
EXPIRE user1 60
```

### Interview Answer

> Fixed Window limits requests in a fixed time period.

---

# 2. Sliding Window

### Definition

Tracks request timestamps dynamically.

### Use When

* API gateways

### Avoid When

* Extremely high request systems

### Advantages / Tradeoffs

**Advantages**

* More accurate

**Tradeoffs**

* More memory

### Limitations

Extra calculations

### Failure scenario / Pitfall

Large timestamp storage

### Common Problems / Fixes

| Problem       | Fix     |
| ------------- | ------- |
| Memory growth | Cleanup |

### Example

```python
ZADD requests
```

### Interview Answer

> Sliding Window tracks request timestamps to provide smoother limits.

---

# 3. Token Bucket

### Definition

Requests consume tokens from bucket.

### Use When

* APIs

### Avoid When

* Strict fixed traffic

### Advantages / Tradeoffs

**Advantages**

* Allows bursts

**Tradeoffs**

* Slight complexity

### Limitations

Needs refill logic

### Failure scenario / Pitfall

Incorrect refill rate

### Common Problems / Fixes

| Problem    | Fix         |
| ---------- | ----------- |
| Bad limits | Tune refill |

### Example

```text
Bucket=100
Request=-1 token
```

### Interview Answer

> Token Bucket allows burst traffic while maintaining average request limits.

---

# Session Store

# 1. Session ID + Redis Hash

### Definition

Stores session data using Redis hash.

### Use When

* Traditional login systems

### Avoid When

* Stateless systems

### Example

```python
redis.hset(
"session:abc",
{"user":101}
)
```

### Interview Answer

> Session IDs with Redis hashes store user session state centrally.

---

# 2. JWT + Redis Blacklist

### Definition

Store revoked JWTs in Redis.

### Use When

* Logout support

### Avoid When

* Small systems

### Example

```python
redis.set(
"blacklist:token",
1
)
```

### Interview Answer

> Redis can track invalid JWTs for logout handling.

---

# 3. Distributed Sessions

### Definition

Multiple servers use shared Redis sessions.

### Use When

* Load-balanced systems

### Avoid When

* Single server

### Example

```text
Load Balancer
      ↓
Server1
Server2
      ↓
Redis
```

### Interview Answer

> Distributed sessions allow all servers to access the same session state.

---

# Pub/Sub

# 1. Basic Pub/Sub

### Definition

Publish messages to channels.

### Use When

* Notifications
* Chat

### Avoid When

* Persistence required

### Example

```python
publish("news","hello")
```

### Interview Answer

> Basic Pub/Sub sends messages to subscribers in real time.

---

# 2. Pattern Pub/Sub

### Definition

Subscribe using patterns.

### Use When

* Multiple channels

### Avoid When

* Fixed channels

### Example

```python
subscribe("news.*")
```

### Interview Answer

> Pattern Pub/Sub allows wildcard subscriptions.

---

# 3. Streams-based Pub/Sub

### Definition

Pub/Sub with persistence and retries.

### Use When

* Production systems

### Avoid When

* Small realtime notifications

### Example

```python
XADD orders
```

### Interview Answer

> Streams-based Pub/Sub provides reliable event delivery with retries and consumer groups.
