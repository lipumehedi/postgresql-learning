# PostgreSQL — Phase 1 Assessment

> **CodeBridge Japan — PostgreSQL Zero to Hero**

A comprehensive Phase 1 assessment covering **Sessions 01–08** of the PostgreSQL learning journey.

This assessment combines conceptual knowledge, practical SQL, debugging, and a small relational database project using **PostgreSQL**.

---

## 📌 Assessment Overview

| Item                    | Details                                                    |
| ----------------------- | ---------------------------------------------------------- |
| **Phase**               | Phase 1 Wrap-up                                            |
| **Sessions Covered**    | 01–08                                                      |
| **Level**               | Absolute Beginner                                          |
| **Database**            | PostgreSQL                                                 |
| **Main Tables**         | `users`, `products`                                        |
| **Assessment Sections** | Knowledge Check · Practical SQL · Debugging · Mini-Project |
| **Mini-Project**        | CodeBridge Feedback System                                 |

---

## 🎯 Assessment Goals

This assessment is designed to verify my understanding of the core PostgreSQL concepts covered in Phase 1.

By completing this assessment, I practiced:

* Database and RDBMS fundamentals
* PostgreSQL and `psql`
* Tables, rows, columns, and schemas
* Primary keys and foreign keys
* Constraints and indexes
* SQL CRUD operations
* Filtering with `WHERE`
* Pattern matching with `LIKE` and `ILIKE`
* Handling `NULL`
* Sorting and pagination
* Aggregate functions
* `GROUP BY` and `HAVING`
* SQL string, numeric, and date/time functions
* `COALESCE` and `CASE`
* Debugging incorrect SQL
* Foreign key relationships
* Basic relational database design

---

# 1. Knowledge Check

## Concepts from Sessions 01–08

### Question 1

**What is the difference between a DBMS and an RDBMS?**

**Answer:**

A **DBMS** is software that manages databases, while an **RDBMS** is a type of DBMS that organizes data into related tables containing rows and columns.

PostgreSQL is an example of an RDBMS.

---

### Question 2

**What SQL statements create and read rows?**

**Answer:**

* `INSERT` creates a new row.
* `SELECT` reads existing rows.

```sql
INSERT INTO users (full_name, email)
VALUES ('Example User', 'example@example.com');

SELECT * FROM users;
```

---

### Question 3

**What are the four CRUD operations?**

| CRUD   | SQL      |
| ------ | -------- |
| Create | `INSERT` |
| Read   | `SELECT` |
| Update | `UPDATE` |
| Delete | `DELETE` |

---

### Question 4

**What is a primary key?**

A primary key is a column, or group of columns, that uniquely identifies each row in a table.

Example:

```sql
id SERIAL PRIMARY KEY
```

Every row must have a unique primary key value so that the row can be identified unambiguously.

---

### Question 5

**What is a foreign key?**

A foreign key creates a relationship between tables.

For example:

```sql
user_id INTEGER REFERENCES users(id)
```

This means that `user_id` must reference an existing `id` in the `users` table.

If the referenced user does not exist, PostgreSQL rejects the insert with a foreign key constraint violation.

---

### Question 6

**What is the difference between a constraint and an index?**

A **constraint** is a rule that PostgreSQL enforces on data.

Examples:

```sql
PRIMARY KEY
NOT NULL
UNIQUE
FOREIGN KEY
```

An **index** is a database structure used to make searches and lookups faster.

A constraint protects data integrity, while an index primarily improves query performance.

---

### Question 7

**Which `psql` commands list tables and describe a table?**

List all tables:

```text
\dt
```

Describe a specific table:

```text
\d users
```

---

### Question 8

**Why is `TIMESTAMPTZ` useful for `created_at`?**

`TIMESTAMPTZ` represents a point in time with time-zone awareness.

It is useful for application timestamps because the same moment can be displayed correctly across different time zones.

Example:

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT now()
```

---

### Question 9

**Why doesn't `WHERE column = NULL` work?**

`NULL` represents an unknown or missing value.

This does not work:

```sql
WHERE phone = NULL
```

Use:

```sql
WHERE phone IS NULL
```

or:

```sql
WHERE phone IS NOT NULL
```

---

### Question 10

**What is the difference between `LIKE` and `ILIKE`?**

`LIKE` performs case-sensitive pattern matching.

```sql
WHERE name LIKE '%keyboard%'
```

`ILIKE` performs case-insensitive pattern matching in PostgreSQL.

```sql
WHERE name ILIKE '%keyboard%'
```

---

### Question 11

**Can row order be relied upon without `ORDER BY`?**

No.

Without `ORDER BY`, PostgreSQL does not guarantee the order in which rows are returned.

Use:

```sql
ORDER BY created_at ASC
```

when a specific order is required.

---

### Question 12

**What is the difference between `WHERE` and `HAVING`?**

`WHERE` filters individual rows **before** grouping.

`HAVING` filters groups **after** aggregation.

Example:

```sql
SELECT date_trunc('month', created_at) AS signup_month,
       COUNT(*) AS total
FROM users
GROUP BY signup_month
HAVING COUNT(*) >= 2;
```

`WHERE` cannot filter `COUNT(*)` because the count does not exist until after grouping and aggregation.

---

### Question 13

**What is the difference between `COUNT(*)` and `COUNT(column_name)`?**

```sql
COUNT(*)
```

counts every row.

```sql
COUNT(column_name)
```

counts only rows where that column is not `NULL`.

---

### Question 14

**What does `COALESCE` do?**

`COALESCE` returns the first non-`NULL` value.

Example:

```sql
SELECT COALESCE(comment, 'No comment left')
FROM feedback;
```

This can provide a user-friendly fallback when optional data is missing.

---

# 2. Practical SQL Test

## Task 1 — Aliases

Return every user's name and email using the aliases `name` and `contact`.

### Solution

```sql
SELECT
    full_name AS name,
    email AS contact
FROM users;
```

---

## Task 2 — Case-Insensitive Search

Find users whose name contains `"Tanaka"` regardless of case.

### Solution

```sql
SELECT *
FROM users
WHERE full_name ILIKE '%tanaka%';
```

Expected result:

```text
Aiko Tanaka
```

---

## Task 3 — Date Filtering

Find users who joined during January or February 2024, ordered from oldest to newest.

### Solution

```sql
SELECT *
FROM users
WHERE created_at >= '2024-01-01'
  AND created_at < '2024-03-01'
ORDER BY created_at ASC;
```

Expected users:

```text
Tanvir Ahmed
Mitsuki Sato
Farzana Rahman
Kenji Watanabe
Rashed Karim
```

---

## Task 4 — Cheapest Products

Return the three cheapest products with prices converted from cents to dollars.

### Solution

```sql
SELECT
    name,
    ROUND(price_cents / 100.0, 2) AS price_dollars
FROM products
ORDER BY price_cents ASC
LIMIT 3;
```

Expected results:

```text
PostgreSQL Cheat Sheet Poster     $9.99
Beginner SQL Handbook            $19.99
Desk Lamp                         $24.99
```

---

## Task 5 — Out-of-Stock Products

Count products where `stock_qty` is zero.

### Solution

```sql
SELECT COUNT(*)
FROM products
WHERE stock_qty = 0;
```

Expected result:

```text
1
```

The out-of-stock product is:

```text
CodeBridge Hoodie
```

---

## Task 6 — Signup Groups

Count users by signup month and show only months with at least two signups.

### Solution

```sql
SELECT
    date_trunc('month', created_at) AS signup_month,
    COUNT(*) AS total
FROM users
GROUP BY signup_month
HAVING COUNT(*) >= 2
ORDER BY signup_month;
```

Expected result:

| Signup Month  | Total |
| ------------- | ----: |
| January 2024  |     2 |
| February 2024 |     3 |
| March 2024    |     2 |

---

## Task 7 — Product Price Labels

Classify products as either:

* `Under $20`
* `$20 and up`

### Solution

```sql
SELECT
    name,
    CASE
        WHEN price_cents / 100.0 < 20
            THEN 'Under $20'
        ELSE '$20 and up'
    END AS price_band
FROM products;
```

---

## Task 8 — Safe UPDATE

Write an `UPDATE` statement that changes Rashed Karim's email.

### Solution

```sql
UPDATE users
SET email = 'rashed.k.new@example.com'
WHERE id = 5;
```

Using the primary key is safer than filtering by name because names are not guaranteed to be unique.

> **Important:** This query should not be run against the original shared seed data unless the database is intentionally being modified for practice.

---

# 3. Debugging Exercises

The goal of this section is to identify **why** a query is wrong, not simply memorize the corrected query.

---

## 🐛 Bug 1 — Comparing with NULL

### Incorrect Query

```sql
SELECT *
FROM users
WHERE phone = NULL;
```

### Problem

`NULL` represents an unknown value and cannot be compared using `=`.

### Correct Query

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

---

## 🐛 Bug 2 — LIMIT Without ORDER BY

### Incorrect Query

```sql
SELECT name, price_cents
FROM products
LIMIT 3;
```

### Problem

`LIMIT 3` returns an arbitrary three rows if no ordering is specified.

### Correct Query

```sql
SELECT
    name,
    price_cents
FROM products
ORDER BY price_cents ASC
LIMIT 3;
```

---

## 🐛 Bug 3 — Integer Division

### Incorrect Query

```sql
SELECT
    name,
    price_cents / 100 AS price_dollars
FROM products;
```

### Problem

Both values are integers, so PostgreSQL performs integer division and can remove the decimal portion.

### Correct Query

```sql
SELECT
    name,
    price_cents / 100.0 AS price_dollars
FROM products;
```

---

## 🐛 Bug 4 — Invalid GROUP BY

### Incorrect Query

```sql
SELECT
    full_name,
    date_trunc('month', created_at),
    COUNT(*)
FROM users
GROUP BY date_trunc('month', created_at);
```

### Problem

`full_name` is neither aggregated nor included in `GROUP BY`.

A month can contain multiple users, so PostgreSQL cannot choose one name for the group.

### Correct Query

```sql
SELECT
    date_trunc('month', created_at) AS signup_month,
    COUNT(*) AS total
FROM users
GROUP BY signup_month;
```

---

## 🐛 Bug 5 — Case-Sensitive Search

### Incorrect Query

```sql
SELECT *
FROM products
WHERE name LIKE '%keyboard%';
```

### Problem

`LIKE` is case-sensitive in PostgreSQL.

`Mechanical Keyboard` does not match lowercase `keyboard`.

### Correct Query

```sql
SELECT *
FROM products
WHERE name ILIKE '%keyboard%';
```

---

## 🐛 Bug 6 — Aggregate in WHERE

### Incorrect Query

```sql
SELECT
    date_trunc('month', created_at) AS signup_month,
    COUNT(*)
FROM users
WHERE COUNT(*) > 2
GROUP BY signup_month;
```

### Problem

`WHERE` runs before grouping and aggregation.

`COUNT(*)` is only available after the groups have been created.

### Correct Query

```sql
SELECT
    date_trunc('month', created_at) AS signup_month,
    COUNT(*) AS total
FROM users
GROUP BY signup_month
HAVING COUNT(*) > 2;
```

---

## 🐛 Bug 7 — Dangerous DELETE

### Incorrect Query

```sql
DELETE FROM users;
```

### Problem

There is no `WHERE` clause.

This deletes **every row** in the table.

### Correct Query

```sql
DELETE FROM users
WHERE id = 999;
```

> ⚠️ Always check the `WHERE` clause carefully before executing `UPDATE` or `DELETE` statements.

---

# 4. Mini-Project — CodeBridge Feedback System

## Project Overview

CodeBridge wants students to leave feedback on products.

Each feedback record contains:

* The user who submitted the feedback
* The product being reviewed
* A rating
* An optional comment
* The creation timestamp

This project introduces a real relational relationship between:

```text
users
   │
   │ user_id
   ▼
feedback
   ▲
   │ product_id
   │
products
```

The `feedback` table references both `users` and `products`.

---

# 5. Create the Feedback Table

Create the table exactly as specified.

```sql
CREATE TABLE feedback (
    id            SERIAL        PRIMARY KEY,
    user_id       INTEGER       NOT NULL REFERENCES users(id),
    product_id    INTEGER       NOT NULL REFERENCES products(id),
    rating        INTEGER       NOT NULL,
    comment       TEXT,
    created_at    TIMESTAMPTZ   NOT NULL DEFAULT now()
);
```

Notice that `comment` does not have `NOT NULL`.

This means a feedback record can exist without a comment.

---

## Verify the Table

Inside `psql`:

```text
\d feedback
```

The table should contain:

| Column       | Type          | Constraint             |
| ------------ | ------------- | ---------------------- |
| `id`         | `SERIAL`      | Primary Key            |
| `user_id`    | `INTEGER`     | Foreign Key + NOT NULL |
| `product_id` | `INTEGER`     | Foreign Key + NOT NULL |
| `rating`     | `INTEGER`     | NOT NULL               |
| `comment`    | `TEXT`        | Nullable               |
| `created_at` | `TIMESTAMPTZ` | NOT NULL + DEFAULT     |

---

# 6. Insert Feedback Data

Insert six feedback records.

```sql
INSERT INTO feedback (user_id, product_id, rating, comment) VALUES
    (1, 1, 5, 'Really clear explanations, helped me a lot.'),
    (2, 1, 4, NULL),
    (3, 2, 3, 'Useful but a bit basic for me.'),
    (4, 4, 5, 'Great build quality.'),
    (5, 4, 4, 'Good keyboard, a bit loud.'),
    (6, 2, 2, NULL);
```

At least two records intentionally contain:

```sql
comment = NULL
```

---

# 7. Test the Foreign Key

Try inserting a feedback row for a user that does not exist.

```sql
INSERT INTO feedback (user_id, product_id, rating)
VALUES (999, 1, 5);
```

Expected error:

```text
ERROR: insert or update on table "feedback" violates foreign key
constraint "feedback_user_id_fkey"

DETAIL:
Key (user_id)=(999) is not present in table "users".
```

### Why did this happen?

`user_id` is a foreign key referencing `users(id)`, but there is no user with `id = 999`.

PostgreSQL therefore rejects the insert to protect referential integrity.

---

# 8. Query High Ratings

Return feedback with ratings of 4 or higher, newest first.

```sql
SELECT *
FROM feedback
WHERE rating >= 4
ORDER BY created_at DESC;
```

---

# 9. Find Feedback Without Comments

Use `IS NULL` to find feedback where no comment was provided.

```sql
SELECT *
FROM feedback
WHERE comment IS NULL;
```

Expected rows:

```text
User 2 → Product 1
User 6 → Product 2
```

---

# 10. Safe UPDATE and DELETE Practice

Create a temporary feedback row:

```sql
INSERT INTO feedback (user_id, product_id, rating, comment)
VALUES (1, 5, 3, 'Practice row');
```

Update it:

```sql
UPDATE feedback
SET comment = 'Updated practice row'
WHERE comment = 'Practice row';
```

Verify:

```sql
SELECT *
FROM feedback
WHERE comment = 'Updated practice row';
```

Delete the temporary row:

```sql
DELETE FROM feedback
WHERE comment = 'Updated practice row';
```

This follows the safe throwaway-row pattern practiced earlier in Phase 1.

---

# 11. Average Rating per Product

Calculate the average rating for each product.

Only include products with at least two feedback records.

```sql
SELECT
    product_id,
    ROUND(AVG(rating), 1) AS avg_rating,
    COUNT(*) AS total_reviews
FROM feedback
GROUP BY product_id
HAVING COUNT(*) >= 2
ORDER BY product_id;
```

Expected result:

| Product ID | Average Rating | Reviews |
| ---------: | -------------: | ------: |
|          1 |            4.5 |       2 |
|          2 |            2.5 |       2 |
|          4 |            4.5 |       2 |

---

# 12. Stretch Challenge — Rating Labels

Add a label based on the average rating.

Rules:

* Below `3` → `Needs improvement`
* `3` through `4` → `Good`
* Above `4` → `Excellent`

### Solution

```sql
SELECT
    product_id,
    ROUND(AVG(rating), 1) AS avg_rating,
    CASE
        WHEN AVG(rating) < 3
            THEN 'Needs improvement'
        WHEN AVG(rating) <= 4
            THEN 'Good'
        ELSE 'Excellent'
    END AS rating_label
FROM feedback
GROUP BY product_id
HAVING COUNT(*) >= 2
ORDER BY product_id;
```

Expected labels:

| Product | Average | Label             |
| ------: | ------: | ----------------- |
|       1 |     4.5 | Excellent         |
|       2 |     2.5 | Needs improvement |
|       4 |     4.5 | Excellent         |

---

# 13. Phase 1 Skills Demonstrated

Completing this assessment demonstrates practical exposure to the following PostgreSQL concepts:

### Database Fundamentals

* Database
* DBMS
* RDBMS
* PostgreSQL
* SQL
* Schemas
* Tables
* Rows
* Columns

### Database Design

* Primary Keys
* Foreign Keys
* Constraints
* Nullable columns
* Data types
* Relationships
* Referential integrity

### SQL

* `CREATE TABLE`
* `INSERT`
* `SELECT`
* `UPDATE`
* `DELETE`
* `WHERE`
* `ORDER BY`
* `GROUP BY`
* `HAVING`
* `LIMIT`
* `OFFSET`

### Filtering

* Comparison operators
* `AND`
* `OR`
* `NOT`
* `IN`
* `BETWEEN`
* `LIKE`
* `ILIKE`
* `IS NULL`
* `IS NOT NULL`

### Aggregation

* `COUNT()`
* `SUM()`
* `AVG()`
* `MIN()`
* `MAX()`
* `GROUP BY`
* `HAVING`

### SQL Functions

* `UPPER()`
* `LOWER()`
* `LENGTH()`
* `TRIM()`
* String concatenation
* `SUBSTRING()`
* `ROUND()`
* `CEIL()`
* `FLOOR()`
* `ABS()`
* `NOW()`
* `EXTRACT()`
* `date_trunc()`
* `AGE()`
* `COALESCE()`
* `NULLIF()`
* `CASE`

### PostgreSQL CLI

```text
\l
\c
\dt
\d
\q
\?
```

---

# 14. Key Lessons Learned

## 1. Always use ORDER BY when order matters

```sql
SELECT *
FROM products
ORDER BY price_cents ASC;
```

Without `ORDER BY`, row order is not guaranteed.

---

## 2. NULL requires special handling

Incorrect:

```sql
WHERE comment = NULL
```

Correct:

```sql
WHERE comment IS NULL
```

---

## 3. WHERE and HAVING have different jobs

```text
WHERE
  ↓
Filter rows
  ↓
GROUP BY
  ↓
Aggregate
  ↓
HAVING
  ↓
Filter groups
```

---

## 4. Be careful with UPDATE and DELETE

Always verify the target rows before modifying data.

```sql
SELECT *
FROM users
WHERE id = 5;
```

Then:

```sql
UPDATE users
SET email = 'new@example.com'
WHERE id = 5;
```

---

## 5. Foreign keys protect relationships

A foreign key prevents records from referencing non-existent parent records.

```sql
REFERENCES users(id)
```

This helps maintain data integrity across related tables.

---

## 6. Use decimal arithmetic for money

Avoid:

```sql
price_cents / 100
```

Prefer:

```sql
price_cents / 100.0
```

when converting integer cents into decimal dollars.

---

# 15. Phase 1 Completion Checklist

* [x] Session 01 — Database Fundamentals
* [x] Session 02 — Core Database Terminology
* [x] Session 03 — PostgreSQL Environment & `psql`
* [x] Session 04 — `CREATE TABLE` & Data Types
* [x] Session 05 — CRUD & `WHERE`
* [x] Session 06 — Sorting & Pagination
* [x] Session 07 — Aggregation
* [x] Session 08 — SQL Functions
* [x] Knowledge Check
* [x] Practical SQL Test
* [x] Debugging Exercises
* [x] Foreign Key Practice
* [x] Mini-Project — CodeBridge Feedback System

---

# 16. Phase 1 Summary

Phase 1 established the foundation for working with PostgreSQL databases.

The assessment moved from basic concepts to practical SQL and finally to a small relational database project.

The most important progression was:

```text
Database Fundamentals
        ↓
Tables & Data Types
        ↓
CRUD Operations
        ↓
Filtering & Sorting
        ↓
Aggregation
        ↓
SQL Functions
        ↓
Debugging
        ↓
Foreign Keys
        ↓
Relational Mini-Project
```

The **CodeBridge Feedback System** brings these concepts together by connecting users, products, and feedback through foreign-key relationships.

---

## 🚀 Next Step

With Phase 1 completed, the next stage is to move beyond individual queries and start working with more realistic database operations and application-style data.

Potential next topics include:

* Multiple-table relationships
* `JOIN`
* One-to-many relationships
* Many-to-many relationships
* Subqueries
* Common Table Expressions
* Advanced filtering
* More complex aggregation
* Database normalization
* Transactions
* Indexes and query performance
* Real-world PostgreSQL projects

---

## 📁 Recommended Repository Structure

```text
postgresql-learning/
│
├── README.md
│
├── Session-01/
│   └── README.md
│
├── Session-02/
│   └── README.md
│
├── Session-03/
│   └── README.md
│
├── Session-04/
│   ├── README.md
│   └── seed_data.sql
│
├── Session-05/
│   └── README.md
│
├── Session-06/
│   └── README.md
│
├── Session-07/
│   └── README.md
│
├── Session-08/
│   └── README.md
│
└── Phase-1-Assessment/
    └── README.md
```

---

## 🧰 Tools Used

* PostgreSQL
* `psql`
* SQL
* Git
* GitHub
* Visual Studio Code

---

## 📚 Learning Context

This assessment is part of my ongoing journey to build practical skills in:

**Python · SQL · PostgreSQL · Git · GitHub · Backend Development**

The goal is to progress from fundamental database concepts toward building practical backend applications and recruiter-ready projects.

---

**Phase 1 — Completed ✅**
