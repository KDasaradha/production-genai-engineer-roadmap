# PostgreSQL

PostgreSQL is the durable storage layer for backend systems, RAG metadata, users, jobs, conversations, audit logs, and product data.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [postgresql-performance.md](postgresql-performance.md) | You need indexes, pagination, query plans, and connection pooling before production | Faster, explainable queries |
| 2 | [postgresql-advanced-patterns.md](postgresql-advanced-patterns.md) | Transactions, locks, isolation, and schema design protect correctness | Production-safe data workflows |

## What To Master

| Area | Why It Matters |
| --- | --- |
| Indexes | Speed up reads but add write and storage cost |
| Query plans | Show why a query is slow |
| Connection pooling | Prevent DB overload from API traffic |
| Transactions | Keep multi-step writes correct |
| Isolation levels | Explain concurrency anomalies |
| Locks | Prevent data races but can create contention |

## Common Trap

Do not say "add an index" as a universal answer. Good answers discuss selectivity, query shape, write cost, storage cost, and query plans.

## Interview Focus

| Question | Strong Answer Should Mention |
| --- | --- |
| How do you debug a slow query? | `EXPLAIN`, indexes, filters, joins, pagination, table size |
| What is a transaction? | Atomic unit of work, commit/rollback, consistency |
| What is connection pooling? | Reuse DB connections, avoid per-request connection overhead |
| When can indexes hurt? | Heavy writes, low selectivity, storage, maintenance |

## Project Connection

Use PostgreSQL in [Knowledge Assistant](../12-Projects/knowledge-assistant.md), [Enterprise RAG Platform](../12-Projects/enterprise-rag-platform.md), and [Production AI Platform](../12-Projects/production-ai-platform.md).
