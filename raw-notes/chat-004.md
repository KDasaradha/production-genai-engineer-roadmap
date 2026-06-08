Databases -- PostgreSQL Indexing, Query Optimization, Transactions, Pooling, Schema Design, Composite Indexes, Row Level Locking, Atomic Transactions Beginner to Advanced

# Databases for Backend Developers (PostgreSQL)

## Beginner → Advanced

These are some of the most important topics for FastAPI/Python backend interviews and real-world systems.

---

# Learning Order

| Level        | Topic               |
| ------------ | ------------------- |
| Beginner     | Schema Design       |
| Beginner     | Indexing            |
| Intermediate | Composite Indexes   |
| Intermediate | Query Optimization  |
| Intermediate | Transactions        |
| Intermediate | Connection Pooling  |
| Advanced     | Atomic Transactions |
| Advanced     | Row-Level Locking   |

---

# 1. Schema Design

## Definition

Schema Design is the process of organizing tables, columns, relationships, and constraints.

A bad schema causes:

* Slow queries
* Data duplication
* Difficult maintenance

A good schema makes everything easier.

---

## Example

### Bad Design

```sql
CREATE TABLE orders (
    id SERIAL,
    customer_name TEXT,
    customer_email TEXT,
    customer_phone TEXT,
    product_name TEXT
);
```

Problems:

```text
Repeated customer data
Repeated product data
Hard to update
```

---

### Good Design

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT UNIQUE
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    product_id INT REFERENCES products(id)
);
```

---

## Use When

Always.

---

## Avoid When

Never avoid proper schema design.

---

# Normalization

## Definition

Reduce duplicate data.

---

### 1NF

No repeated columns.

Bad:

```text
phone1
phone2
phone3
```

Good:

```text
customer_phones table
```

---

### 2NF

Every column depends on full primary key.

---

### 3NF

No indirect dependencies.

Bad:

```text
Customer
City
State
Country
```

State depends on City.

Move to separate table.

---

# Denormalization

Sometimes we intentionally duplicate data for speed.

Example:

```sql
orders.total_amount
```

Instead of recalculating every time.

---

# Interview Answer

> Schema design is the process of organizing database tables and relationships while balancing normalization and performance requirements.

---

# 2. Indexing

---

## What Is Index?

Imagine a book.

Without index:

```text
Search page by page
```

With index:

```text
Jump directly to page
```

Database index works exactly the same way.

---

## Without Index

```sql
SELECT *
FROM users
WHERE email='abc@gmail.com';
```

Database scans entire table.

```text
O(n)
```

---

## With Index

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Now:

```text
O(log n)
```

---

## Internal Working

PostgreSQL primarily uses:

### B-Tree Index

Structure:

```text
        M
      /   \
     F     T
```

Like a sorted tree.

---

## Example

```sql
CREATE INDEX idx_users_name
ON users(name);
```

---

## Check Query Plan

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE name='John';
```

Shows:

```text
Index Scan
```

or

```text
Sequential Scan
```

---

## Advantages

* Faster reads
* Faster searches

---

## Tradeoff

Slower inserts/updates.

Because index must also update.

---

# 3. Composite Indexes

---

## Definition

Index on multiple columns.

---

Example:

```sql
CREATE INDEX idx_user_status
ON users(country, status);
```

---

Table:

| country | status   |
| ------- | -------- |
| India   | Active   |
| India   | Inactive |
| USA     | Active   |

---

Works Well

```sql
WHERE country='India'
```

Works Well

```sql
WHERE country='India'
AND status='Active'
```

---

Not Good

```sql
WHERE status='Active'
```

Because index order matters.

---

### Rule

```text
Left-most Prefix Rule
```

Index:

```sql
(country, status)
```

Can use:

```text
country
country + status
```

Cannot efficiently use:

```text
status only
```

---

# Interview Answer

> Composite indexes are indexes created on multiple columns and are most effective when query filters follow the left-most column order.

---

# 4. Query Optimization

---

## Goal

Make queries faster.

---

## Example 1

Bad:

```sql
SELECT *
FROM users;
```

Good:

```sql
SELECT id,name
FROM users;
```

Fetch only needed columns.

---

## Example 2

Bad

```sql
SELECT *
FROM orders
WHERE LOWER(email)='abc@gmail.com';
```

Index cannot be used.

---

Good

```sql
SELECT *
FROM orders
WHERE email='abc@gmail.com';
```

---

## Use EXPLAIN

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email='abc@gmail.com';
```

Shows:

```text
Execution Time
Rows
Cost
Index Scan
```

---

## Common Optimization Techniques

### Proper Indexes

### Avoid SELECT *

### Reduce JOINs

### Use Pagination

Bad:

```sql
SELECT *
FROM orders;
```

Good:

```sql
LIMIT 100 OFFSET 0;
```

---

# 5. Transactions

---

## Definition

A transaction is a group of operations treated as a single unit.

---

Example

Transfer ₹1000.

```text
Deduct from A
Add to B
```

Both must succeed.

---

## SQL

```sql
BEGIN;

UPDATE accounts
SET balance=balance-1000
WHERE id=1;

UPDATE accounts
SET balance=balance+1000
WHERE id=2;

COMMIT;
```

---

If error:

```sql
ROLLBACK;
```

---

# ACID Properties

---

## A — Atomicity

All or nothing.

---

## C — Consistency

Data remains valid.

---

## I — Isolation

Transactions don't interfere.

---

## D — Durability

Committed data survives crashes.

---

# Interview Answer

> Transactions ensure multiple operations execute safely using ACID guarantees.

---

# 6. Atomic Transactions

---

## Definition

Operations succeed together or fail together.

---

Example

Order Placement

```text
Create Order
Reduce Inventory
Create Payment Record
```

If one fails:

```text
Everything rollback
```

---

Example

```sql
BEGIN;

INSERT INTO orders(...);

UPDATE inventory
SET quantity=quantity-1;

INSERT INTO payments(...);

COMMIT;
```

---

# FastAPI Example

```python
async with session.begin():

    order = Order(...)

    session.add(order)

    inventory.quantity -= 1

    payment = Payment(...)

    session.add(payment)
```

Everything commits together.

---

# 7. Row-Level Locking

---

## Problem

Two users withdrawing simultaneously.

Balance:

```text
1000
```

User A reads:

```text
1000
```

User B reads:

```text
1000
```

Both withdraw:

```text
700
```

Result:

```text
-400
```

Wrong.

---

## Solution

Lock row.

```sql
SELECT *
FROM accounts
WHERE id=1
FOR UPDATE;
```

Now:

```text
Transaction A gets lock
Transaction B waits
```

---

## Example

```sql
BEGIN;

SELECT *
FROM accounts
WHERE id=1
FOR UPDATE;

UPDATE accounts
SET balance=balance-500;

COMMIT;
```

---

## Use Cases

### Wallets

### Banking

### Inventory

### Ticket Booking

---

# Interview Answer

> Row-level locking prevents concurrent transactions from modifying the same row simultaneously, avoiding race conditions.

---

# 8. Connection Pooling

---

## Problem

Creating DB connection every request is expensive.

---

Without Pool

```text
Request
Open Connection
Execute Query
Close Connection
```

Thousands of times.

---

## With Pool

```text
Request
Borrow Connection
Execute Query
Return Connection
```

---

## Benefits

* Faster requests
* Less DB load
* Better scalability

---

## FastAPI + SQLAlchemy

```python
engine = create_async_engine(
    DATABASE_URL,
    pool_size=20,
    max_overflow=10
)
```

---

## Pool Settings

| Setting      | Meaning                     |
| ------------ | --------------------------- |
| pool_size    | Base connections            |
| max_overflow | Extra temporary connections |
| pool_timeout | Wait time                   |
| pool_recycle | Refresh old connections     |

---

# Real Interview Scenario

Imagine building a Food Delivery App.

### Place Order API

Uses:

✅ Transaction

```text
Create Order
Create Payment
Reduce Inventory
```

✅ Atomicity

```text
All succeed or rollback
```

✅ Row Locking

```text
Prevent overselling inventory
```

✅ Indexing

```text
Search orders quickly
```

✅ Composite Index

```sql
(user_id, status)
```

✅ Query Optimization

```text
Fast dashboard queries
```

✅ Pooling

```text
Thousands of concurrent requests
```

---

# Senior-Level Topics You Should Learn Next

After mastering these, move to:

1. Database Isolation Levels
2. Deadlocks
3. VACUUM & ANALYZE
4. PostgreSQL MVCC
5. Partitioning
6. Replication
7. Read Replicas
8. Sharding
9. Materialized Views
10. CTEs (Common Table Expressions)
11. Window Functions
12. JSONB
13. Full Text Search
14. Database Migrations (Alembic)
15. PostgreSQL Internals

These are the topics typically expected from strong backend engineers working with FastAPI and PostgreSQL.


---

# PostgreSQL Advanced Topics (Part 2)

## From Intermediate → Senior Backend Engineer

These topics are heavily used in production systems and frequently appear in senior backend interviews.

---

# 1. Database Isolation Levels

## Why Do We Need Isolation?

Imagine:

```text
Transaction A
Transaction B
```

running simultaneously.

Without proper isolation:

* Dirty Reads
* Lost Updates
* Non-repeatable Reads
* Phantom Reads

can occur.

---

## Example

Account Balance:

```text
1000
```

Transaction A:

```sql
UPDATE accounts
SET balance = 500
WHERE id = 1;
```

But has NOT committed yet.

Transaction B reads:

```sql
SELECT balance
FROM accounts;
```

Should B see:

```text
1000
or
500 ?
```

Isolation levels determine that behavior.

---

## PostgreSQL Isolation Levels

### Read Committed (Default)

Most common.

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

Behavior:

```text
Only committed data is visible.
```

---

### Repeatable Read

Guarantees same result during transaction.

Example:

Transaction A:

```sql
BEGIN;

SELECT balance FROM accounts;
```

returns:

```text
1000
```

Transaction B:

```sql
UPDATE accounts
SET balance=2000;
COMMIT;
```

Transaction A reads again:

```sql
SELECT balance FROM accounts;
```

still gets:

```text
1000
```

---

### Serializable

Highest isolation.

Database behaves as if transactions run one-by-one.

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

Most expensive.

Used in:

* Banking
* Financial systems

---

# Interview Question

### Which isolation level does PostgreSQL use by default?

Answer:

```text
READ COMMITTED
```

---

# 2. Deadlocks

## What Is Deadlock?

Two transactions waiting for each other forever.

---

Example

Transaction A:

```sql
UPDATE accounts
SET balance=100
WHERE id=1;
```

Locks row 1.

---

Transaction B:

```sql
UPDATE accounts
SET balance=100
WHERE id=2;
```

Locks row 2.

---

Now:

Transaction A:

```sql
UPDATE accounts
SET balance=100
WHERE id=2;
```

Waits.

---

Transaction B:

```sql
UPDATE accounts
SET balance=100
WHERE id=1;
```

Waits.

---

Result:

```text
Deadlock
```

---

PostgreSQL detects this automatically:

```text
ERROR:
deadlock detected
```

One transaction is killed.

---

## Prevention

Always lock rows in same order.

Bad:

```text
A -> 1 then 2
B -> 2 then 1
```

Good:

```text
A -> 1 then 2
B -> 1 then 2
```

---

# 3. MVCC (Multi-Version Concurrency Control)

One of PostgreSQL's most important features.

---

## Problem

How can reads happen while writes occur?

Other databases often use heavy locking.

PostgreSQL uses:

```text
MVCC
```

---

## Idea

Instead of changing row directly:

```text
Old Row
New Row
```

Both versions exist temporarily.

---

Example

Current row:

```text
Balance = 1000
```

Transaction A updates:

```text
Balance = 500
```

PostgreSQL creates:

```text
Version 1 = 1000
Version 2 = 500
```

Readers see appropriate version.

---

Benefits:

✅ Less locking

✅ High concurrency

✅ Fast reads

---

# Interview Question

### Why is PostgreSQL fast for concurrent workloads?

Answer:

```text
Because of MVCC.
Readers don't block writers and writers don't block readers.
```

---

# 4. VACUUM

Because MVCC creates old row versions.

Those old rows must eventually be removed.

---

Example

Update same row:

```sql
UPDATE users ...
UPDATE users ...
UPDATE users ...
```

Many dead versions accumulate.

---

VACUUM cleans them.

```sql
VACUUM users;
```

---

### Autovacuum

PostgreSQL automatically runs:

```text
Autovacuum
```

in background.

Usually sufficient.

---

# VACUUM FULL

```sql
VACUUM FULL users;
```

Also reclaims disk space.

But:

```text
Locks table
```

Use carefully.

---

# ANALYZE

Updates query planner statistics.

```sql
ANALYZE users;
```

Allows PostgreSQL to choose:

```text
Index Scan
or
Sequential Scan
```

correctly.

---

# Interview Answer

> VACUUM removes dead tuples created by MVCC, while ANALYZE updates planner statistics.

---

# 5. Partitioning

## Problem

Huge tables.

Example:

```text
orders
```

contains:

```text
500 million rows
```

Queries become slower.

---

## Solution

Split table into partitions.

---

Example

Orders by year.

```sql
CREATE TABLE orders (
    id BIGINT,
    created_at DATE
)
PARTITION BY RANGE(created_at);
```

---

Partitions:

```sql
CREATE TABLE orders_2024
PARTITION OF orders
FOR VALUES FROM ('2024-01-01')
TO ('2025-01-01');
```

---

```sql
CREATE TABLE orders_2025
PARTITION OF orders
FOR VALUES FROM ('2025-01-01')
TO ('2026-01-01');
```

---

Query:

```sql
SELECT *
FROM orders
WHERE created_at='2025-05-01';
```

PostgreSQL only checks:

```text
orders_2025
```

Not all partitions.

---

Benefits:

* Faster queries
* Easier maintenance

---

# 6. Replication

## Problem

Database overloaded by reads.

---

Example

```text
100 writes/sec
10000 reads/sec
```

---

Solution:

```text
Primary DB
    |
    |
Read Replicas
```

---

Primary:

```text
Handles writes
```

Replica:

```text
Handles reads
```

---

Applications:

```python
write_db.execute(...)
read_db.execute(...)
```

---

Benefits

```text
Scale Reads
```

without scaling writes.

---

# Types

### Streaming Replication

Most common.

### Logical Replication

Replicate selected tables.

---

# 7. Read Replicas

Special case of replication.

---

Architecture

```text
          App
           |
   -----------------
   |               |
Write DB      Read Replica
```

---

Example

Instagram Feed

```text
Reads >> Writes
```

Perfect use case.

---

Queries

```sql
SELECT posts ...
```

go to replica.

---

Writes

```sql
INSERT post ...
```

go to primary.

---

# Limitation

Replication lag.

---

Primary:

```text
New row inserted
```

Replica may receive it:

```text
1 second later
```

---

Need to design around eventual consistency.

---

# 8. Sharding

## Problem

Single database cannot scale anymore.

---

Example

```text
5 TB database
```

Too large.

---

Split data.

---

Shard 1

```text
Users 1-1M
```

Shard 2

```text
Users 1M-2M
```

Shard 3

```text
Users 2M-3M
```

---

Routing:

```python
user_id % 3
```

determines shard.

---

Benefits:

```text
Unlimited scaling
```

---

Drawbacks:

```text
Complex joins
Complex transactions
Complex backups
```

---

# Interview Answer

> Partitioning splits tables inside a database, while sharding splits data across multiple databases.

---

# 9. Materialized Views

Normal View:

```sql
CREATE VIEW active_users AS
SELECT *
FROM users
WHERE active=true;
```

Runs query every time.

---

Materialized View:

```sql
CREATE MATERIALIZED VIEW sales_report AS
SELECT
    region,
    SUM(amount)
FROM sales
GROUP BY region;
```

Stores results physically.

---

Query:

```sql
SELECT *
FROM sales_report;
```

Very fast.

---

Refresh:

```sql
REFRESH MATERIALIZED VIEW sales_report;
```

---

Use Cases

* Dashboards
* Analytics
* Reports

---

# 10. CTE (Common Table Expressions)

Makes complex queries readable.

---

Without CTE

```sql
SELECT ...
FROM (
   SELECT ...
) t;
```

---

With CTE

```sql
WITH high_salary AS (
    SELECT *
    FROM employees
    WHERE salary > 100000
)
SELECT *
FROM high_salary;
```

---

Benefits

* Readability
* Reusability

---

# 11. Window Functions

One of PostgreSQL's most powerful features.

---

Example

Employee Salary Ranking.

---

Data

| Name | Salary |
| ---- | ------ |
| A    | 100    |
| B    | 200    |
| C    | 300    |

---

Query

```sql
SELECT
    name,
    salary,
    RANK() OVER(
        ORDER BY salary DESC
    ) AS rank
FROM employees;
```

Result

| Name | Salary | Rank |
| ---- | ------ | ---- |
| C    | 300    | 1    |
| B    | 200    | 2    |
| A    | 100    | 3    |

---

Common Functions

```sql
RANK()
ROW_NUMBER()
DENSE_RANK()
LEAD()
LAG()
SUM()
AVG()
```

---

# 12. JSONB

PostgreSQL can store JSON efficiently.

---

Example

```sql
CREATE TABLE products(
    id SERIAL,
    metadata JSONB
);
```

---

Insert

```sql
INSERT INTO products(metadata)
VALUES (
'{
 "color":"black",
 "size":"XL"
}'
);
```

---

Query

```sql
SELECT *
FROM products
WHERE metadata->>'color'='black';
```

---

Can index JSONB.

```sql
CREATE INDEX idx_products_json
ON products
USING GIN(metadata);
```

---

Use Cases

* Dynamic fields
* Configurations
* Metadata

---

# 13. Full Text Search

Search inside text.

---

Example

```sql
SELECT *
FROM articles
WHERE to_tsvector(content)
@@ plainto_tsquery('postgres');
```

---

Finds:

```text
PostgreSQL
postgres
postgres database
```

---

Much better than:

```sql
LIKE '%postgres%'
```

---

# 14. Database Migrations (Alembic)

Never manually edit production schema.

Use migrations.

---

Example

Model Change:

```python
class User(Base):
    age = Column(Integer)
```

Generate migration:

```bash
alembic revision --autogenerate
```

Apply:

```bash
alembic upgrade head
```

Rollback:

```bash
alembic downgrade -1
```

---

# 15. PostgreSQL Internals (Senior Level)

Understand:

### Query Parser

```text
SQL → Parse Tree
```

### Planner

Chooses:

```text
Index Scan
Bitmap Scan
Seq Scan
```

### Executor

Runs plan.

---

Example

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email='abc@gmail.com';
```

Execution pipeline:

```text
Parser
 ↓
Planner
 ↓
Executor
```

---

# What Interviewers Expect

### Junior Backend

* CRUD
* Indexes
* Basic Transactions

### Mid-Level Backend

* Composite Indexes
* Query Optimization
* Locking
* Pooling

### Senior Backend

* MVCC
* Isolation Levels
* Deadlocks
* Partitioning
* Replication
* Read Replicas
* Window Functions
* JSONB
* PostgreSQL Internals

---

## Next PostgreSQL Topics Worth Learning

1. Advanced EXPLAIN ANALYZE
2. Covering Indexes (`INCLUDE`)
3. Partial Indexes
4. GIN vs B-Tree vs GiST Indexes
5. BRIN Indexes
6. Database Performance Tuning
7. PgBouncer
8. Connection Pooling Internals
9. PostgreSQL High Availability
10. PostgreSQL Disaster Recovery

These are the topics that separate a typical backend developer from a strong PostgreSQL-focused backend engineer.

---

Yes. I can teach PostgreSQL from beginner to advanced, including the topics expected from Junior, Mid-Level, Senior, and Staff-level backend engineers.

Since you're targeting Python/FastAPI backend roles, I would focus on **practical PostgreSQL for production systems**, not just SQL syntax.

# PostgreSQL Learning Roadmap

| Level           | Topics                                          |
| --------------- | ----------------------------------------------- |
| Beginner        | Database Fundamentals, SQL Basics, Tables, CRUD |
| Intermediate    | Joins, Constraints, Indexes, Views              |
| Advanced        | Transactions, Locking, Query Optimization       |
| Production      | Pooling, Partitioning, Replication, Migrations  |
| Senior Engineer | Schema Design, Performance Tuning, Scaling      |
| Expert          | Internals, MVCC, WAL, Vacuum, Query Planner     |

---

# Module 1: Database Fundamentals

Before PostgreSQL, understand:

## What is a Database?

A database is a system used to store and manage data.

Example:

Instead of storing users in JSON:

```json
[
    {
        "id":1,
        "name":"Krishna"
    }
]
```

Store them in a table:

| id | name    |
| -- | ------- |
| 1  | Krishna |

Benefits:

* Fast search
* Data consistency
* Security
* Relationships
* Transactions

---

# What is PostgreSQL?

PostgreSQL is an open-source relational database.

Features:

* ACID compliant
* Highly reliable
* Supports JSON
* Full Text Search
* Advanced Indexing
* Transactions
* Extensions

Companies using PostgreSQL:

* Instagram
* Reddit
* Stripe
* GitLab

---

# Relational Database Concepts

A PostgreSQL database contains:

```text
Database
 ├── Tables
 │     ├── Rows
 │     └── Columns
 │
 ├── Views
 ├── Indexes
 ├── Functions
 └── Triggers
```

---

# Table

A table stores data.

Example:

```sql
CREATE TABLE users (
    id INTEGER,
    name VARCHAR(100),
    email VARCHAR(255)
);
```

---

# Row

Single record.

```text
id | name
-----------
1  | Krishna
```

One row = one user.

---

# Column

A field in table.

```text
id
name
email
```

---

# Primary Key

Uniquely identifies records.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```

Example:

```text
1
2
3
```

Cannot repeat.

---

# Data Types

Most common types:

```sql
INTEGER
BIGINT
TEXT
VARCHAR
BOOLEAN
DATE
TIMESTAMP
JSON
JSONB
UUID
```

Example:

```sql
CREATE TABLE users (
    id UUID,
    name TEXT,
    age INTEGER,
    is_active BOOLEAN
);
```

---

# Create Database

```sql
CREATE DATABASE company_db;
```

Connect:

```sql
\c company_db
```

---

# Create Table

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);
```

---

# CRUD Operations

## Insert

```sql
INSERT INTO users(name,email)
VALUES
('Krishna','k@gmail.com');
```

---

## Select

```sql
SELECT * FROM users;
```

Output:

```text
id | name
---------
1  | Krishna
```

---

## Select Specific Columns

```sql
SELECT name,email
FROM users;
```

---

## Where

```sql
SELECT *
FROM users
WHERE id = 1;
```

---

## Update

```sql
UPDATE users
SET name='KK'
WHERE id=1;
```

---

## Delete

```sql
DELETE FROM users
WHERE id=1;
```

---

# Constraints

Rules for data.

## NOT NULL

```sql
name TEXT NOT NULL
```

Cannot be empty.

---

## UNIQUE

```sql
email TEXT UNIQUE
```

No duplicates.

---

## PRIMARY KEY

```sql
id SERIAL PRIMARY KEY
```

Unique + Not Null.

---

## CHECK

```sql
age INTEGER CHECK(age > 18)
```

---

## DEFAULT

```sql
created_at TIMESTAMP DEFAULT NOW()
```

---

# Exercise

Create:

```sql
users
```

Table with:

```text
id
name
email
age
created_at
```

Requirements:

* id → Primary Key
* email → Unique
* age → greater than 18
* created_at → current timestamp

Try writing it yourself before looking up solutions.

---

# Module 2 Preview (Next Lesson)

We'll go deeper into:

1. Foreign Keys
2. One-to-One
3. One-to-Many
4. Many-to-Many
5. Joins
6. Inner Join
7. Left Join
8. Right Join
9. Full Join
10. Self Join
11. Aggregate Functions
12. Group By
13. Having
14. Real-world Ecommerce Schema

These are the foundation for every FastAPI + PostgreSQL application.

After that we'll move into the topics most developers struggle with:

* Indexing
* Composite Indexes
* Query Optimization
* Transactions
* MVCC
* Row-Level Locking
* Connection Pooling
* Schema Design
* Partitioning
* Replication
* WAL
* Vacuum
* Performance Tuning

Those are the topics that separate a junior developer from a senior backend engineer.


---

Absolutely. Given your goal (Python/FastAPI backend jobs), I would teach PostgreSQL as if you were preparing for:

* Backend Developer interviews
* FastAPI + PostgreSQL production applications
* Senior Backend Engineer concepts
* Real-world system design discussions

# How We Will Learn PostgreSQL

We'll follow this order:

| Phase | Topics                    |
| ----- | ------------------------- |
| 1     | Database Fundamentals     |
| 2     | SQL Basics                |
| 3     | Relationships             |
| 4     | Joins                     |
| 5     | Aggregations              |
| 6     | Constraints               |
| 7     | Views                     |
| 8     | Indexing                  |
| 9     | Transactions              |
| 10    | Locking                   |
| 11    | Query Optimization        |
| 12    | Schema Design             |
| 13    | JSON & JSONB              |
| 14    | Partitioning              |
| 15    | Connection Pooling        |
| 16    | Replication               |
| 17    | PostgreSQL Internals      |
| 18    | Production Best Practices |

---

# Phase 1 — Database Fundamentals

Before PostgreSQL, you must understand **why databases exist**.

---

## Problem Without a Database

Imagine an e-commerce application.

You store users in a file:

```json
[
  {
    "id": 1,
    "name": "Krishna",
    "email": "kk@gmail.com"
  }
]
```

Problems:

### Searching

Find user:

```python
for user in users:
    if user["email"] == email:
        return user
```

Time complexity:

```text
O(n)
```

Database:

```sql
SELECT *
FROM users
WHERE email='kk@gmail.com';
```

Much faster because of indexes.

---

### Concurrency

Suppose:

User A updates file.

User B updates file.

Who wins?

Data may be lost.

Database solves this using:

* Transactions
* Locks
* MVCC

We'll learn these later.

---

### Relationships

Suppose:

```text
User
Order
Product
Payment
```

Files become messy.

Databases organize relationships naturally.

---

# What is a Database?

A database is software that stores and retrieves data efficiently.

Examples:

| Database   | Type       |
| ---------- | ---------- |
| PostgreSQL | Relational |
| MySQL      | Relational |
| SQLite     | Relational |
| MongoDB    | Document   |
| Redis      | Key Value  |

---

# What is PostgreSQL?

PostgreSQL is:

* Open source
* Free
* Extremely powerful
* ACID compliant
* Production ready

Many large companies use it.

It supports:

* SQL
* JSON
* Transactions
* Full Text Search
* Extensions
* Partitioning
* Replication

---

# Understanding Relational Databases

A relational database stores data in tables.

Example:

## Users Table

| id | name    |
| -- | ------- |
| 1  | Krishna |
| 2  | John    |

---

## Orders Table

| id | user_id | amount |
| -- | ------- | ------ |
| 1  | 1       | 100    |
| 2  | 1       | 200    |

Notice:

```text
user_id
```

connects orders to users.

This is called a relationship.

---

# Database → Table → Row → Column

Think of:

```text
Database
    ↓
Tables
    ↓
Rows
    ↓
Columns
```

Example:

## users table

| id | name    | email                               |
| -- | ------- | ----------------------------------- |
| 1  | Krishna | [kk@gmail.com](mailto:kk@gmail.com) |

---

### Row

One record.

```text
1 | Krishna | kk@gmail.com
```

is a row.

---

### Column

```text
id
name
email
```

are columns.

---

# Primary Key

Every table should identify records uniquely.

Example:

| id | name    |
| -- | ------- |
| 1  | Krishna |
| 2  | John    |

`id` is unique.

SQL:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```

---

## Why Primary Keys Matter

Without PK:

```text
Krishna
Krishna
Krishna
```

Which row should be updated?

Impossible to know.

With PK:

```text
1 Krishna
2 Krishna
3 Krishna
```

Now every row is unique.

---

# Data Types

Every column stores a specific type.

---

## Integer

```sql
age INTEGER
```

Example:

```text
20
35
50
```

---

## Bigint

```sql
views BIGINT
```

Large numbers.

---

## Text

```sql
name TEXT
```

Unlimited length strings.

---

## Varchar

```sql
email VARCHAR(255)
```

Limited length strings.

---

## Boolean

```sql
is_active BOOLEAN
```

Values:

```text
true
false
```

---

## Date

```sql
birth_date DATE
```

Example:

```text
2026-01-01
```

---

## Timestamp

```sql
created_at TIMESTAMP
```

Example:

```text
2026-01-01 10:30:00
```

---

## UUID

Used heavily in modern applications.

Example:

```text
550e8400-e29b-41d4-a716-446655440000
```

SQL:

```sql
id UUID PRIMARY KEY
```

---

## JSONB

One of PostgreSQL's superpowers.

```sql
preferences JSONB
```

Example:

```json
{
  "theme": "dark",
  "language": "en"
}
```

---

# Creating a Database

```sql
CREATE DATABASE ecommerce;
```

Connect:

```sql
\c ecommerce
```

---

# Creating Your First Table

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);
```

---

# CRUD Operations

CRUD means:

```text
Create
Read
Update
Delete
```

---

## CREATE

Insert data.

```sql
INSERT INTO users (
    name,
    email
)
VALUES (
    'Krishna',
    'kk@gmail.com'
);
```

---

## READ

```sql
SELECT *
FROM users;
```

Result:

| id | name    | email                               |
| -- | ------- | ----------------------------------- |
| 1  | Krishna | [kk@gmail.com](mailto:kk@gmail.com) |

---

## Specific Columns

```sql
SELECT name,email
FROM users;
```

---

## Filtering

```sql
SELECT *
FROM users
WHERE id = 1;
```

---

## UPDATE

```sql
UPDATE users
SET name='KK'
WHERE id=1;
```

---

## DELETE

```sql
DELETE
FROM users
WHERE id=1;
```

---

# Important Interview Question

### Difference Between DELETE, TRUNCATE, DROP?

## DELETE

Removes rows.

```sql
DELETE FROM users;
```

Table remains.

---

## TRUNCATE

Removes all rows quickly.

```sql
TRUNCATE TABLE users;
```

Table remains.

---

## DROP

Removes entire table.

```sql
DROP TABLE users;
```

Table disappears.

---

# Exercise 1 (Do This Yourself)

Create a table called:

```text
employees
```

Requirements:

| Column     | Type      |
| ---------- | --------- |
| id         | SERIAL    |
| name       | TEXT      |
| email      | TEXT      |
| salary     | INTEGER   |
| is_active  | BOOLEAN   |
| created_at | TIMESTAMP |

Rules:

* id → primary key
* email → unique
* salary > 0
* created_at default current timestamp

Try it without looking up the answer.

---

# What You'll Learn Next

In the next lesson we'll cover **Relationships & Foreign Keys**, including:

* One-to-One
* One-to-Many
* Many-to-Many
* Foreign Keys
* Cascading Deletes
* Real-world Ecommerce Schema

These concepts are used in almost every FastAPI + PostgreSQL application and are asked frequently in interviews.
