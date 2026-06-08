# PostgreSQL Advanced Patterns

## Definition

This file consolidates the unique PostgreSQL material from the raw chat notes that extends the existing performance note: schema design, normalization, denormalization, transactions, atomic operations, row-level locking, isolation levels, deadlocks, and MVCC.

## Why It Exists

`postgresql-performance.md` covers indexing, optimization, and pooling. Production backends also need correctness under concurrency, not only speed.

## When To Use

- Use these patterns in financial flows, order placement, inventory, user accounts, ticketing, and any workload where consistency matters.
- Use denormalization selectively when read performance justifies duplication.
- Use row locks and transactions when multiple requests can modify the same business entity.

## When Not To Use

- Do not add locks or serializable isolation everywhere by default.
- Do not denormalize blindly without understanding write amplification and stale data risk.
- Do not rely on application-only checks for critical consistency constraints.

## Internal Working

| Pattern | Problem Solved | Main Tradeoff |
| --- | --- | --- |
| Normalization | Duplicate data and maintenance pain | More joins |
| Denormalization | Expensive read aggregation | More write complexity |
| Transaction | Multi-step consistency | Longer lock time |
| Row-level lock | Lost updates and overselling | Blocking and contention |
| Isolation level | Read/write anomalies | Higher isolation can reduce throughput |
| MVCC | Readers blocked by writers | Vacuum and storage overhead |

## Key Topics

### Schema Design

- Normalize core entities first.
- Separate customers, products, and orders rather than repeating fields.
- Add constraints early: primary keys, foreign keys, uniqueness, and not-null rules.

### Transactions and Atomicity

- Wrap business operations that must succeed together inside a transaction.
- Examples: create order, decrement inventory, create payment record.
- Use rollback on failure instead of partially committing business state.

### Row-Level Locking

- Use `SELECT ... FOR UPDATE` when concurrent requests can update the same row.
- Typical cases: wallet balance, stock reservation, coupon redemption, ticket booking.

### Isolation Levels

| Level | What It Prevents Better | Typical Use |
| --- | --- | --- |
| Read Committed | Dirty reads | Default application workloads |
| Repeatable Read | Non-repeatable reads | Multi-step analytical or transactional reads |
| Serializable | Most anomalies | Banking or very strict correctness paths |

### Deadlocks

- They happen when transactions acquire locks in inconsistent order.
- Prevent them by locking rows in the same sequence across code paths.
- Keep transactions small and short-lived.

### MVCC

- PostgreSQL uses multi-version concurrency control so readers can continue while writers update rows.
- This improves concurrency but introduces vacuum and bloat management concerns.

## Common Mistakes

- Treating indexes as a replacement for good schema design.
- Keeping transactions open across slow network calls.
- Forgetting that high isolation has a throughput cost.
- Locking too broadly and creating unnecessary contention.
- Ignoring soft delete, audit fields, and pagination in production APIs.

## Best Practices

- Separate performance tuning from correctness design, then combine both deliberately.
- Use idempotency keys for retry-prone APIs like payments.
- Keep transactions short and avoid remote calls inside them.
- Add `EXPLAIN ANALYZE` to your debugging workflow.
- Review lock behavior in any workflow with concurrent writes.

## Interview Questions

1. What is the difference between normalization and denormalization?
2. When would you use `SELECT ... FOR UPDATE`?
3. What isolation level does PostgreSQL use by default?
4. What is MVCC and why is it useful?
5. How do deadlocks happen and how do you reduce them?

## Interview Answers

- Normalization reduces duplication and keeps data consistent; denormalization duplicates some data intentionally to improve reads.
- `SELECT ... FOR UPDATE` is used when concurrent transactions must not update the same row independently.
- PostgreSQL defaults to `READ COMMITTED`.
- MVCC lets PostgreSQL serve readers without forcing them to wait on most writes by keeping row versions.
- Deadlocks happen when transactions wait on each other in conflicting lock order; prevent them with consistent ordering and shorter transactions.

## Related Topics

- [postgresql-performance.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/03-PostgreSQL/postgresql-performance.md)
- [fastapi-advanced.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/02-FastAPI/fastapi-advanced.md)
- [concurrency-models.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/01-Python/concurrency-models.md)
