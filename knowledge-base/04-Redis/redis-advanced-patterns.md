# Redis Advanced Patterns

## Definition

This file consolidates the Redis material that was not fully represented in the curated backend note: data structures, cache strategies, eviction, TTL, rate limiting, Pub/Sub, Streams, distributed locks, Cluster, Sentinel, sessions, Celery usage, transactions, and production pitfalls.

## Why It Exists

The base Redis note covers caching and AI backend optimization. Production backend and GenAI systems also use Redis for coordination, reliability, and traffic control.

## When To Use

- Use Redis for hot reads, short-lived data, rate limiting, sessions, queues, pub/sub style broadcast, and lightweight coordination.
- Use Redis Streams when events must be durable and replayable.
- Use Redis Cluster when one node no longer fits memory or throughput.
- Use Sentinel when high availability matters more than sharding.

## When Not To Use

- Do not use Redis as the only source of truth for critical durable business records.
- Do not use Pub/Sub when delivery guarantees are required.
- Do not use write-behind caching where data loss is unacceptable.
- Do not use a distributed lock without expiration and failure handling.

## Internal Working

| Area | Use Case | Tradeoff |
| --- | --- | --- |
| Cache-aside | Read-heavy APIs | First miss is slow |
| Write-through | Fresh cache on writes | Slower write path |
| Write-behind | Very fast writes | Risk of data loss |
| Fixed window limit | Simplicity | Boundary burst problem |
| Sliding window limit | Better fairness | More memory and logic |
| Pub/Sub | Real-time transient fan-out | No persistence |
| Streams | Durable event processing | More operational complexity |
| Distributed lock | Prevent duplicate processing | Must handle expiry safely |

## Core Patterns

### Data Structures

- Strings for cache values and counters.
- Hashes for object-like fields.
- Lists for simple queues.
- Sets for uniqueness.
- Sorted sets for ranking and sliding-window rate limiting.
- Streams for durable event logs.

### Cache Pitfalls

- Cache stampede: many misses trigger the same expensive work.
- Cache penetration: invalid requests bypass cache and hit the database.
- Cache avalanche: many keys expire together and overload the origin.
- Fixes: randomized TTL, null caching, per-key locking, prewarming, and layered caching.

### Rate Limiting

| Strategy | Best For | Main Risk |
| --- | --- | --- |
| Fixed window | Simple quotas | Burst at boundaries |
| Sliding window | Fair API rate limits | More memory and sorting work |
| Token bucket | Allowing controlled bursts | Token refill tuning |
| Leaky bucket | Smoothing traffic | Less burst flexibility |

### Messaging and Background Work

- Pub/Sub fits live dashboards, chat notifications, and transient broadcasts.
- Streams fit durable background processing, consumer groups, replay, and ordered events.
- Celery with Redis is useful for background jobs but requires operational monitoring and idempotent tasks.

### Availability and Scale

- Sentinel gives failover and service discovery.
- Cluster gives sharding plus high availability.
- They solve different problems and should not be treated as interchangeable.

## Common Mistakes

- Forgetting TTL on temporary data.
- Storing large blobs under hot keys.
- Treating Pub/Sub as a reliable queue.
- Leaving distributed locks without expiry.
- Using Redis only as a cache but ignoring rate limiting, sessions, or coordination opportunities.

## Best Practices

- Keep Redis responsibilities explicit: cache, session, queue, or coordination.
- Add connection pooling and timeouts at the client layer.
- Track hit rate, memory usage, eviction rate, and latency.
- Use idempotent job processing when queues or streams are involved.
- Document which keys are critical, temporary, or reconstructable.

## Interview Questions

1. Why Redis instead of database caching?
2. Cache-aside vs write-through vs write-behind?
3. Pub/Sub vs Streams?
4. Sentinel vs Cluster?
5. Why use sorted sets for sliding-window rate limiting?

## Interview Answers

- Redis keeps data in memory and supports multiple fast data structures, which makes it better for hot-path caching and coordination than a disk-based database alone.
- Cache-aside is simplest and most common; write-through keeps cache fresh at write time; write-behind is fast but risks durability gaps.
- Pub/Sub is transient broadcast messaging, while Streams are durable, replayable, and support consumer groups.
- Sentinel handles high availability and failover; Cluster handles sharding plus high availability.
- Sorted sets let you use timestamps as scores, remove expired entries efficiently, and count requests in a moving time window.

## Related Topics

- [redis-and-caching.md](redis-and-caching.md)
- [fastapi-advanced.md](../02-FastAPI/fastapi-advanced.md)
- [backend-and-ai-scenarios.md](../05-System-Design/backend-and-ai-scenarios.md)
