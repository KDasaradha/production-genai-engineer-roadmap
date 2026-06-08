# Redis Interview Questions

## Questions

1. Why use Redis instead of only a database cache table?
Answer: Redis is in-memory, low-latency, and supports rich data structures for caching, rate limiting, sessions, and queues.

2. Cache-aside vs write-through?
Answer: Cache-aside populates on read miss; write-through updates cache during writes for fresher cache state.

3. Pub/Sub vs Streams?
Answer: Pub/Sub is transient broadcast; Streams are durable, replayable, and support consumer groups.

4. Sentinel vs Cluster?
Answer: Sentinel handles failover and service discovery; Cluster adds sharding and horizontal scaling.

5. Why do sliding-window rate limiters often use sorted sets?
Answer: Sorted sets store timestamps as scores, which makes expiring old entries and counting recent requests efficient.

## Related Topics

- [redis-and-caching.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/04-Redis/redis-and-caching.md)
- [redis-advanced-patterns.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/04-Redis/redis-advanced-patterns.md)
