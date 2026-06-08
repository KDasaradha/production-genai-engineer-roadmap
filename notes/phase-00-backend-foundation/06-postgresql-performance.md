# PostgreSQL Performance

## 1. Problem Statement

PostgreSQL performance solves the problem of slow queries, poor indexing, inconsistent data, and overloaded connections.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | PostgreSQL performance is about modeling, indexing, querying, and pooling data efficiently. |
| Use When | Your app stores relational data or metadata. |
| Avoid When | You need only temporary cache data. |
| Advantages | Reliable transactions and powerful querying. |
| Tradeoffs | Requires schema and index design. |
| Limitations | Not ideal for pure vector similarity without extensions or a vector DB. |
| Example | Index `user_id` for fast lookup. |
| Production Example | Store document metadata, users, permissions, and chat sessions. |
| Interview Answer | PostgreSQL performance depends on schema design, indexes, query plans, transactions, and connection management. |

## 3. Intermediate Explanation

Core components include tables, indexes, query planner, transactions, locks, and connection pools.

## 4. Advanced Explanation

Use `EXPLAIN ANALYZE`, partial indexes, composite indexes, careful transaction scopes, and pooling.

## 5. Internal Working

```text
SQL query -> planner -> index/table scan -> locks/transactions -> result
```

## 6. When To Use

Use for durable relational data, metadata, audit logs, users, roles, sessions, and document records.

## 7. When NOT To Use

Avoid using plain PostgreSQL as the only store for large-scale semantic retrieval unless using the right extension and indexes.

## 8. Advantages

Strong consistency, rich queries, mature tooling, and transaction safety.

## 9. Tradeoffs

Indexes speed reads but slow writes and consume storage.

## 10. Limitations

Bad queries can still overload the database.

## 11. Real-World Examples

RAG metadata store, chat history, tenant permissions, billing events.

## 12. Architecture Diagram

```text
[FastAPI] -> [Repository] -> [Connection Pool] -> [PostgreSQL]
```

## 13. Python Implementation

```python
def build_document_query(tenant_id: str) -> tuple[str, dict[str, str]]:
    return "select * from documents where tenant_id = :tenant_id", {"tenant_id": tenant_id}
```

## 14. FastAPI Implementation

Use a dependency to create request-scoped sessions and keep route handlers thin.

## 15. Database Integration

Use indexes on high-cardinality filters, foreign keys for integrity, and migrations for schema changes.

## 16. Production Considerations

Monitor slow queries, connection pool saturation, locks, deadlocks, and migration risk.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | No indexes | Index common filters |
| Intermediate | Indexing everything | Index based on queries |
| Production | Too many DB connections | Use pooling and limits |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is an index? | A data structure that speeds lookup. |
| Intermediate | Composite index order? | It should match common filter and sort patterns. |
| Advanced | How debug a slow query? | Use `EXPLAIN ANALYZE`, indexes, query rewrite, and metrics. |
| Scenario | RAG app is slow. | Check metadata filters, indexes, pool saturation, and vector retrieval separately. |

## 19. System Design Discussion

PostgreSQL is the source of truth for business data around AI workflows.

## 20. Hands-On Assignment

- Easy: Create a table and index.
- Medium: Compare query plans.
- Hard: Design RAG metadata tables.

## 21. Mini Project

Build document metadata storage for a RAG app.

## 22. Production-Level Project

Build multi-tenant document storage with permissions, audit logs, and optimized queries.

## Quiz

1. What is an index?
2. Why not index every column?
3. What is a transaction?
4. What is connection pooling?
5. How do you debug slow queries?
6. What is `EXPLAIN ANALYZE`?
7. What data should PostgreSQL store in a RAG system?
8. What causes lock contention?
9. Why are migrations risky?
10. When use a vector DB instead?

## Knowledge Check

You should be able to explain indexes, transactions, pooling, and PostgreSQL's role in AI systems.

Are you ready for the next section?

---

# Day 7: PostgreSQL Optimization + Indexing + Connection Pooling + Production Database Patterns

This topic becomes important once applications start growing.

Most beginner apps work with:

```text
10 users
```

Production systems may handle:

```text
1000+
10,000+
100,000+ users
```

Poor database design can turn:

```text
50ms query
```

into:

```text
5–10 seconds
```

---

# Part 1: How PostgreSQL searches data

Suppose table:

```text
users

id | username | email
------------------------
1  | KK       | kk@gmail.com
2  | John     | john@gmail.com
3  | Mike     | mike@gmail.com
...
1000000 rows
```

Query:

```sql
SELECT * 
FROM users
WHERE username='KK';
```

Without optimization:

```text
Start row 1
↓
Check row 2
↓
Check row 3
↓
...
↓
Find result
```

This is:

```text
Full Table Scan
```

Very slow on large datasets.

---

# Part 2: Indexes

Indexes work like a book index.

Book without index:

```text
Find chapter "Redis"

↓

Page 1
Page 2
Page 3
...
```

Book with index:

```text
Redis → Page 126
```

Database index:

```sql
CREATE INDEX idx_username
ON users(username);
```

Now:

```sql
SELECT *
FROM users
WHERE username='KK';
```

becomes much faster.

---

# Common index types

| Type      | Use              |
| --------- | ---------------- |
| B-Tree    | equality, ranges |
| Hash      | exact match      |
| Composite | multiple columns |
| Unique    | uniqueness       |
| Full-text | searching text   |

---

# Example: Unique index

```sql
CREATE UNIQUE INDEX idx_email
ON users(email);
```

Prevents:

```text
abc@gmail.com
abc@gmail.com
```

duplicates.

---

# Composite index

Suppose:

```sql
SELECT *
FROM users
WHERE country='India'
AND age=25;
```

Instead of:

```sql
CREATE INDEX idx_country;
CREATE INDEX idx_age;
```

Use:

```sql
CREATE INDEX idx_country_age
ON users(country,age);
```

---

# Important ordering rule

```sql
(country,age)
```

Works well for:

```sql
WHERE country='India'
```

and:

```sql
WHERE country='India'
AND age=25
```

Less useful for:

```sql
WHERE age=25
```

Column order matters.

---

# Part 3: Query optimization

Bad:

```sql
SELECT *
FROM users;
```

Why bad?

Because:

```text
Returns every column
```

Better:

```sql
SELECT id,username
FROM users;
```

---

Bad:

```sql
SELECT *
FROM orders
WHERE YEAR(created_at)=2025;
```

Why bad?

Function on column prevents index usage.

Better:

```sql
SELECT *
FROM orders
WHERE created_at
BETWEEN
'2025-01-01'
AND
'2025-12-31';
```

---

# Pagination

Bad:

```sql
SELECT *
FROM users;
```

Good:

```sql
SELECT *
FROM users
LIMIT 10 OFFSET 0;
```

---

FastAPI example:

```python
@app.get("/users")
async def get_users(
    page:int=1,
    size:int=10
):

    offset=(page-1)*size

    return (
        db.query(User)
        .offset(offset)
        .limit(size)
        .all()
    )
```

---

# Part 4: Connection Pooling

Bad:

```python
@app.get("/users")
async def users():

    db=Database()

    users=db.query()

    db.close()
```

For:

```text
1000 users
```

you create:

```text
1000 DB connections
```

Expensive.

---

Better:

```text
Connection Pool

↓

Reuse existing connections
```

Visual:

```text
Request 1 → Connection A

Request 2 → Connection B

Request 3 → Connection A reused
```

---

# SQLAlchemy example

```python
from sqlalchemy import create_engine

engine=create_engine(

    DATABASE_URL,

    pool_size=10,

    max_overflow=20,

    pool_timeout=30
)
```

Meaning:

| Parameter    | Meaning                     |
| ------------ | --------------------------- |
| pool_size    | base connections            |
| max_overflow | extra temporary connections |
| pool_timeout | waiting time                |

---

# Database Dependency Pattern

```python
from sqlalchemy.orm import Session

def get_db():

    db=SessionLocal()

    try:

        yield db

    finally:

        db.close()
```

Route:

```python
@app.get("/users")
async def users(
    db:Session=Depends(get_db)
):

    return db.query(User).all()
```

---

# Part 5: Production Database Patterns

## Soft delete

Bad:

```sql
DELETE FROM users
WHERE id=5;
```

Data disappears forever.

Better:

Add:

```sql
is_deleted BOOLEAN
```

Update:

```sql
UPDATE users
SET is_deleted=true
WHERE id=5;
```

Query:

```sql
SELECT *
FROM users
WHERE is_deleted=false;
```

---

## Created and Updated timestamps

SQLAlchemy:

```python
from sqlalchemy import Column
from sqlalchemy import DateTime
from datetime import datetime

created_at=Column(
    DateTime,
    default=datetime.utcnow
)

updated_at=Column(
    DateTime,
    onupdate=datetime.utcnow
)
```

---

## UUID IDs

Bad:

```text
1
2
3
4
```

Users can guess IDs.

Better:

```text
3f55b7ae-xxxx-xxxx
```

SQLAlchemy:

```python
import uuid

id=Column(
    UUID,
    primary_key=True,
    default=uuid.uuid4
)
```

---

# Production AI backend flow

```text
Client

↓

FastAPI

↓

Middleware

↓

Auth

↓

Redis Cache

↓

DB Connection Pool

↓

PostgreSQL

↓

Background Workers
```

---

# Practice Exercise

### 1. Create index

For:

```sql
products

id
name
category
price
```

Optimize:

```sql
SELECT *
FROM products
WHERE category='Laptop'
```

---

### 2. Add pagination

Expected:

```text
/users?page=2&size=20
```

---

### 3. Add soft delete

Expected:

```text
DELETE user

↓

is_deleted=True
```

---

# Interview Focus

### Q1: Why use indexes?

Indexes speed up searching by avoiding full table scans.

---

### Q2: Why avoid `SELECT *`?

Because it fetches unnecessary data.

---

### Q3: Why use connection pooling?

To reuse DB connections and improve performance.

---

### Q4: Why use soft delete?

To avoid permanent data loss.

---

### Q5: Why use UUIDs?

* Harder to guess
* Better security
* Useful for distributed systems

---

# Mini Project Task

Build:

```text
GET /chat-history
```

Requirements:

* PostgreSQL table
* Pagination
* Connection pooling
* Index on `user_id`
* Soft delete
* timestamps

Next lesson:

**Day 8 → Docker + Docker Compose + containerizing FastAPI + PostgreSQL + Redis + production deployment patterns**
