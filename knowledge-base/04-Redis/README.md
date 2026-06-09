# Redis

Redis is the low-latency support layer for caching, rate limiting, sessions, queues, locks, and streaming workflows.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [redis-and-caching.md](redis-and-caching.md) | Caching and TTLs are the first Redis use cases for AI APIs | Response cache and rate limiter |
| 2 | [redis-advanced-patterns.md](redis-advanced-patterns.md) | Pub/Sub, Streams, locks, and clusters support production workflows | Queue or stream-backed worker pattern |

## What To Master

| Area | Why It Matters |
| --- | --- |
| Caching | Reduces latency and model/API cost |
| TTLs | Prevent stale data and unbounded memory growth |
| Rate limiting | Protects APIs and LLM budgets |
| Pub/Sub | Good for live notifications but not durable processing |
| Streams | Better for durable event processing |
| Distributed locks | Useful but easy to misuse |

## Common Trap

Do not use Redis as a replacement for PostgreSQL when you need durable, relational, auditable data.

## Interview Focus

| Question | Strong Answer Should Mention |
| --- | --- |
| When use Redis? | Cache, sessions, counters, rate limits, queues, locks |
| What is cache invalidation? | Expiry, write-through/write-around, stale reads, consistency tradeoffs |
| Pub/Sub vs Streams? | Pub/Sub is ephemeral; Streams can retain messages and support consumers |
| What can go wrong? | Stale cache, memory pressure, lock misuse, hot keys |

## Project Connection

Use Redis in [AI Streaming Chat API](../12-Projects/ai-streaming-chat-api.md), [AI Security Gateway](../12-Projects/ai-security-gateway.md), and [Production AI Platform](../12-Projects/production-ai-platform.md).
