# Session 08 — Index and Query Performance Basics

**CodeBridge Japan — PostgreSQL Learning**

## Session Overview

| Topic         | Details                                              |
| ------------- | ---------------------------------------------------- |
| Module        | Relationship and Real Queries                        |
| Session       | 08 / 08                                              |
| Prerequisites | Session 07 — Constraint Details                      |
| Level         | Beginner — Intermediate                              |
| Main Topics   | Indexes, EXPLAIN, EXPLAIN ANALYZE, Query Performance |

---

## 1. What Is an Index?

An **index** is an additional data structure that PostgreSQL uses to find rows more efficiently.

Think of a book's index: instead of reading every page to find a topic, you can look up the topic and go directly to the relevant page.

Similarly, a database index helps PostgreSQL locate matching rows without necessarily scanning the entire table.

### Creating an Index

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

Indexes are useful for columns frequently used in:

* `WHERE` conditions
* `JOIN` operations
* Certain `ORDER BY` operations

However, indexes also require storage and add overhead to `INSERT`, `UPDATE`, and `DELETE` operations.

---

## 2. Why Are Indexes Necessary?

### Without an Index

PostgreSQL may need to examine every row in a table to find a matching value.

This is called a **Sequential Scan (Seq Scan)**.

For a small table, this is often fast enough. But for a large table containing millions of rows, scanning every row can become expensive.

### With an Index

PostgreSQL can use an index to locate matching rows more efficiently.

This can reduce the amount of data PostgreSQL needs to examine, especially when a query returns only a small number of rows.

**Important:** PostgreSQL does not always use an index. For small tables or queries that return many rows, a sequential scan may be more efficient.

---

## 3. Seq Scan vs. Index Scan

### Sequential Scan

PostgreSQL reads the table's rows and checks which ones match the query condition.

```text
Seq Scan
   |
   v
Read rows one by one
   |
   v
Check each row
   |
   v
Return matching rows
```

### Index Scan

PostgreSQL searches the index for matching entries and then retrieves the corresponding table rows.

```text
Index Scan
   |
   v
Search the index
   |
   v
Locate matching row references
   |
   v
Retrieve matching rows
```

### Key Difference

| Feature        | Seq Scan                               | Index Scan                                 |
| -------------- | -------------------------------------- | ------------------------------------------ |
| Data examined  | May examine every table row            | Uses index entries to locate matching rows |
| Small tables   | Often efficient                        | May not be necessary                       |
| Large tables   | Can be expensive for selective lookups | Can improve selective lookups              |
| Storage        | No separate index required             | Requires an index                          |
| Write overhead | No index maintenance for this scan     | Index must be maintained during writes     |

---

## 4. CREATE INDEX

### 4.1 Basic Syntax

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

### 4.2 Creating an Index on a Foreign Key Column

PostgreSQL automatically creates an index for a primary key or unique constraint.

However, it does **not** automatically create an index on the referencing side of a foreign key.

For example:

```sql
CREATE INDEX idx_orders_user_id
ON orders (user_id);
```

This index can help queries that filter or join using `orders.user_id`.

### Why Index Foreign Key Columns?

Foreign key columns are often used in `JOIN` operations.

Indexes on these columns may improve query performance and can also help PostgreSQL locate referencing rows when a parent row is updated or deleted.

---

## 5. Understanding EXPLAIN

The `EXPLAIN` command shows the execution plan PostgreSQL chooses for a query.

It helps answer questions such as:

* Will PostgreSQL use a sequential scan or an index?
* Which table will be scanned?
* What operations will be performed?
* How much work does PostgreSQL estimate the query will require?

### 5.1 Basic Syntax

```sql
EXPLAIN
SELECT *
FROM table_name
WHERE column_name = value;
```

### 5.2 Example: Searching by Email

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'aiko.tanaka@example.com';
```

### Example Output

```text
Seq Scan on users (cost=0.00..1.09 rows=1 width=64)
  Filter: (email = 'aiko.tanaka@example.com'::text)
```

### Understanding the Output

| Term       | Meaning                                               |
| ---------- | ----------------------------------------------------- |
| `Seq Scan` | PostgreSQL scans the table sequentially.              |
| `cost`     | Estimated relative work, not elapsed time in seconds. |
| `rows`     | Estimated number of rows returned by the plan node.   |
| `width`    | Estimated average row size in bytes.                  |
| `Filter`   | Condition used to filter rows.                        |

The exact plan and estimates depend on the database, table size, statistics, and PostgreSQL version.

---

## 6. EXPLAIN ANALYZE

`EXPLAIN ANALYZE` executes the query and displays the execution plan together with actual runtime information.

### Syntax

```sql
EXPLAIN ANALYZE
SELECT *
FROM table_name
WHERE column_name = value;
```

### Example

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'aiko.tanaka@example.com';
```

### Important Difference

| Command           | Behavior                                                        |
| ----------------- | --------------------------------------------------------------- |
| `EXPLAIN`         | Shows the estimated execution plan without executing the query. |
| `EXPLAIN ANALYZE` | Executes the query and reports actual execution details.        |

**Caution:** `EXPLAIN ANALYZE` really runs the query. Be careful when using it with statements that modify data, such as `UPDATE` or `DELETE`.

---

## 7. Primary Key Index

PostgreSQL automatically creates a unique index for a primary key.

For example:

```sql
EXPLAIN
SELECT *
FROM users
WHERE id = 3;
```

### Example Output

```text
Index Scan using users_pkey on users
  (cost=0.15..8.17 rows=1 width=64)
  Index Condition: (id = 3)
```

### Explanation

* `Index Scan` means PostgreSQL is using an index to locate matching rows.
* `users_pkey` is the index associated with the primary key.
* `Index Condition` shows the condition used to search the index.

No additional index is needed for the primary key lookup.

---

## 8. Real-World Example — Measuring Index Performance

Rafi wants to compare query performance before and after creating an index.

To avoid changing the original database schema, he creates a temporary test table with 100,000 rows.

### Step 1: Create a Test Table

```sql
CREATE TABLE temp_big_test (
    id SERIAL PRIMARY KEY,
    value INTEGER
);

INSERT INTO temp_big_test (value)
SELECT generate_series(1, 100000);
```

This creates 100,000 rows with values from `1` to `100000`.

### Step 2: Search Without an Index

```sql
EXPLAIN ANALYZE
SELECT *
FROM temp_big_test
WHERE value = 54321;
```

### Example Output

```text
Seq Scan on temp_big_test
  (cost=0.00..1791.00 rows=1 width=8)
  (actual time=8.912..15.331 rows=1 loops=1)
  Filter: (value = 54321)
  Rows Removed by Filter: 99999
```

The sequential scan examines the table and removes rows that do not match the condition.

### Step 3: Create an Index

```sql
CREATE INDEX idx_temp_value
ON temp_big_test (value);
```

### Step 4: Run the Query Again

```sql
EXPLAIN ANALYZE
SELECT *
FROM temp_big_test
WHERE value = 54321;
```

### Example Output

```text
Index Scan using idx_temp_value on temp_big_test
  (cost=0.29..8.31 rows=1 width=8)
  (actual time=0.045..0.047 rows=1 loops=1)
  Index Cond: (value = 54321)
```

### Performance Comparison

The example outputs show how the query plan can change after adding an index.

| Metric                | Without Index | With Index |
| --------------------- | ------------: | ---------: |
| Plan                  |      Seq Scan | Index Scan |
| Estimated total cost  |       1791.00 |       8.31 |
| Actual execution time |     15.331 ms |   0.047 ms |
| Rows returned         |             1 |          1 |

These are illustrative measurements from the supplied example, not guaranteed results. Actual performance depends on hardware, cache state, PostgreSQL settings, and data distribution.

### Step 5: Clean Up the Test Table

```sql
DROP TABLE temp_big_test;
```

This removes the temporary test table and its index.

---

## 9. Adding Indexes to the Phase 2 Schema

Throughout Phase 2, we used several foreign key columns in joins.

The following indexes can help queries that frequently filter or join using these columns.

### 9.1 Index on orders.user_id

```sql
CREATE INDEX idx_orders_user_id
ON orders (user_id);
```

### 9.2 Index on order_items.order_id

```sql
CREATE INDEX idx_order_items_order_id
ON order_items (order_id);
```

### 9.3 Index on order_items.product_id

```sql
CREATE INDEX idx_order_items_product_id
ON order_items (product_id);
```

### 9.4 Index on payments.order_id

```sql
CREATE INDEX idx_payments_order_id
ON payments (order_id);
```

### 9.5 Index on products.category_id

```sql
CREATE INDEX idx_products_category_id
ON products (category_id);
```

### All Indexes Together

```sql
CREATE INDEX idx_orders_user_id
ON orders (user_id);

CREATE INDEX idx_order_items_order_id
ON order_items (order_id);

CREATE INDEX idx_order_items_product_id
ON order_items (product_id);

CREATE INDEX idx_payments_order_id
ON payments (order_id);

CREATE INDEX idx_products_category_id
ON products (category_id);
```

**Note:** Before creating indexes, check whether an equivalent index already exists. Duplicate or unnecessary indexes consume storage and increase write overhead.

---

## 10. Practice Exercises

### Exercise 1 — Create an Index

1. Create an index on `users.email`.
2. Run an `EXPLAIN` query that searches for a specific email.
3. Observe whether PostgreSQL uses the index.

### Exercise 2 — Understand the Primary Key Index

1. Run `EXPLAIN` on a query filtering by `users.id`.
2. Identify the primary key index in the execution plan.
3. Explain why another index is not required for the same primary key lookup.

### Exercise 3 — Compare Query Plans

1. Create a test table with many rows.
2. Run `EXPLAIN ANALYZE` before creating an index.
3. Create an index on the search column.
4. Run `EXPLAIN ANALYZE` again.
5. Compare the plan and actual execution time.

### Exercise 4 — Index Foreign Key Columns

1. Create an index on `orders.user_id`.
2. Create an index on `order_items.order_id`.
3. Run a query that joins these tables.
4. Inspect the execution plan.

---

## 11. Key Concepts Learned

* What a database index is and how it helps locate rows.
* The difference between Sequential Scan and Index Scan.
* How to create indexes using `CREATE INDEX`.
* Why PostgreSQL automatically indexes primary keys but not referencing foreign key columns.
* How to read query plans using `EXPLAIN`.
* How to measure actual query execution using `EXPLAIN ANALYZE`.
* How to interpret estimated cost, actual time, rows, and filters.
* How indexes can improve selective queries.
* Why indexes also add storage and write overhead.
* How to identify useful indexes for frequently used foreign key columns.

---

## 12. Key Takeaway

**Indexes can improve query performance, but they should be created based on actual query patterns and execution plans.**

Use `EXPLAIN` to understand PostgreSQL's plan and `EXPLAIN ANALYZE` to measure actual execution.

The goal is not to add an index to every column. The goal is to choose indexes that help frequently used queries without adding unnecessary overhead.

---

## 13. Learning Progress

### Phase 2 — Relationship and Real Queries

* [x] Session 01 — Multi-Table Schemas & Foreign Keys
* [x] Session 02 — INNER JOIN
* [x] Session 03 — LEFT JOIN, RIGHT JOIN & FULL JOIN
* [x] Session 04 — JOIN, WHERE, GROUP BY & Aggregate Functions
* [x] Session 05 — Subqueries
* [x] Session 06 — Transactions
* [x] Session 07 — Constraint Details (UNIQUE, CHECK, ON DELETE)
* [x] Session 08 — Index and Query Performance Basics

---

## Phase 2 Completed

Congratulations! You have completed all eight sessions of Phase 2.

You have practiced multi-table relationships, joins, aggregate queries, subqueries, transactions, constraints, and query performance basics.

**Next Step:** Review the Phase 2 concepts and prepare for the next learning phase.
