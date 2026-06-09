# PostgreSQL Interview Questions

## Questions

1. Why do indexes improve reads but hurt writes?
Answer: Indexes give faster lookup paths, but they also need maintenance on insert, update, and delete operations.

2. What is a composite index and why does order matter?
Answer: It indexes multiple columns together, and PostgreSQL uses the left-most prefix most effectively.

3. What are ACID properties?
Answer: Atomicity, Consistency, Isolation, and Durability are the guarantees that protect multi-step transactional correctness.

4. When do you use row-level locking?
Answer: When concurrent requests can modify the same row and create lost updates or overselling.

5. What is MVCC?
Answer: Multi-Version Concurrency Control lets readers and writers proceed with less blocking by keeping row versions.

## Related Topics

- [postgresql-performance.md](../03-PostgreSQL/postgresql-performance.md)
- [postgresql-advanced-patterns.md](../03-PostgreSQL/postgresql-advanced-patterns.md)
