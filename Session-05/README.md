# Session 05 — CRUD & WHERE

> **Module:** Working with Data
> **Sheet:** 05 / 08
> **Level:** Absolute Beginner
> **Prerequisite:** Session 04 — CREATE TABLE & Data Types
> **Database:** PostgreSQL

---

## 🎯 Learning Objectives

By the end of this session, I will be able to:

* Understand the four CRUD operations.
* Insert new rows using `INSERT`.
* Retrieve data using `SELECT`.
* Rename output columns using aliases.
* Filter rows using `WHERE`.
* Use comparison operators and logical conditions.
* Filter data using `IN`, `BETWEEN`, `LIKE`, and `ILIKE`.
* Check for missing values using `IS NULL` and `IS NOT NULL`.
* Update and delete specific rows safely.

---

## 01 — What Is CRUD?

**CRUD** stands for the four basic operations used to manage data in an application.

| Operation | SQL Statement | Purpose                |
| --------- | ------------- | ---------------------- |
| Create    | `INSERT`      | Add a new row          |
| Read      | `SELECT`      | Retrieve existing rows |
| Update    | `UPDATE`      | Modify existing data   |
| Delete    | `DELETE`      | Remove a row           |

A **column alias** is a temporary name used to label a column in query results. It does not change the actual table structure.

A **WHERE clause** filters the rows a SQL statement applies to.

> ⚠️ Without a `WHERE` clause, `UPDATE` and `DELETE` can affect every row in the table.

---

## 02 — Why Do We Need CRUD?

Almost every backend feature relies on one or more CRUD operations.

| Application Feature | CRUD Operation |
| ------------------- | -------------- |
| User sign-up        | Create         |
| View user profile   | Read           |
| Change password     | Update         |
| Delete user account | Delete         |

The `WHERE` clause helps target the correct records.

For example, updating one user's email and updating every user's email use the same `UPDATE` statement structure. The difference is whether a condition is included.

---

## 03 — How WHERE Works

PostgreSQL evaluates the `WHERE` condition for each row. Only rows that satisfy the condition are returned or affected.

Example:

```sql
SELECT full_name
FROM users
WHERE id <= 3;
```

Expected result:

| ID | Full Name      | Condition |
| -: | -------------- | --------- |
|  1 | Tanvir Ahmed   | `TRUE`    |
|  2 | Mitsuki Sato   | `TRUE`    |
|  3 | Farzana Rahman | `TRUE`    |

Rows with IDs 4–7 do not satisfy the condition and are excluded.

**Result:** 3 rows returned.

`WHERE` filters the result of a query; it does not modify the table.

---

## 04 — SQL Syntax & Filtering Tools

### 4.1 INSERT — Create a Row

```sql
INSERT INTO table_name (column1, column2)
VALUES (value1, value2);
```

### 4.2 SELECT — Read Data

```sql
-- Select specific columns
SELECT column1, column2
FROM table_name;

-- Select every column
SELECT *
FROM table_name;

-- Use column aliases
SELECT column1 AS alias1, column2 AS alias2
FROM table_name;
```

`*` means all columns.

### 4.3 WHERE — Comparison Operators

```sql
SELECT *
FROM table_name
WHERE column = value;
```

| Operator     | Meaning                  |
| ------------ | ------------------------ |
| `=`          | Equal to                 |
| `<>` or `!=` | Not equal to             |
| `<`          | Less than                |
| `>`          | Greater than             |
| `<=`         | Less than or equal to    |
| `>=`         | Greater than or equal to |

### 4.4 AND / OR / NOT

```sql
-- Both conditions must be true
SELECT *
FROM table_name
WHERE condition1 AND condition2;

-- At least one condition must be true
SELECT *
FROM table_name
WHERE condition1 OR condition2;

-- Negate a condition
SELECT *
FROM table_name
WHERE NOT condition1;
```

### 4.5 IN — Match Values in a List

```sql
SELECT *
FROM table_name
WHERE column IN (value1, value2, value3);
```

### 4.6 BETWEEN — Match an Inclusive Range

```sql
SELECT *
FROM table_name
WHERE column BETWEEN low AND high;
```

`BETWEEN` includes both boundary values.

### 4.7 LIKE / ILIKE — Pattern Matching

```sql
-- Case-sensitive matching
SELECT *
FROM table_name
WHERE column LIKE 'pattern';

-- Case-insensitive matching in PostgreSQL
SELECT *
FROM table_name
WHERE column ILIKE 'pattern';
```

Pattern symbols:

* `%` — Matches any sequence of characters, including zero characters.
* `_` — Matches exactly one character.

### 4.8 IS NULL / IS NOT NULL

```sql
-- Find missing values
SELECT *
FROM table_name
WHERE column IS NULL;

-- Find non-missing values
SELECT *
FROM table_name
WHERE column IS NOT NULL;
```

**Note:** Every column in the current `users` table is `NOT NULL`, so `IS NULL` will not match any row. It becomes useful when optional columns are introduced.

### 4.9 UPDATE — Modify Existing Rows

```sql
UPDATE table_name
SET column = new_value
WHERE condition;
```

### 4.10 DELETE — Remove Rows

```sql
DELETE FROM table_name
WHERE condition;
```

---

## 05 — Basic SQL Examples

These examples use the `users` table created in Session 04.

### Column Aliases

```sql
SELECT
    full_name AS name,
    email AS contact
FROM users;
```

### Comparison Operator

```sql
SELECT full_name
FROM users
WHERE id > 5;
```

### AND

```sql
SELECT full_name
FROM users
WHERE id > 2 AND id < 6;
```

### IN

```sql
SELECT full_name
FROM users
WHERE id IN (1, 3, 5);
```

### BETWEEN

```sql
SELECT full_name
FROM users
WHERE id BETWEEN 2 AND 4;
```

### ILIKE

```sql
SELECT full_name
FROM users
WHERE email ILIKE '%SATO%';
```

This returns Mitsuki Sato because `ILIKE` ignores letter case.

### UPDATE — Illustration Only

```sql
UPDATE users
SET full_name = 'Tanvir H. Ahmed'
WHERE id = 1;
```

### DELETE — Illustration Only

```sql
DELETE FROM users
WHERE id = 1;
```

> ⚠️ Do not run the last two examples against the original seed data. They would modify or remove a row used by later sessions. Use the safe practice workflow in Section 07 instead.

---

## 06 — Realistic Example: Find Students Who Joined in February

Suppose CodeBridge staff want to find all students who joined during February 2024.

```sql
SELECT full_name, created_at
FROM users
WHERE created_at >= '2024-02-01'
  AND created_at < '2024-03-01';
```

### Expected Result

| Full Name      | Created At             |
| -------------- | ---------------------- |
| Farzana Rahman | 2024-02-02 08:05:00+09 |
| Kenji Watanabe | 2024-02-10 11:47:00+09 |
| Rashed Karim   | 2024-02-14 19:22:00+09 |

This query selects records from February 1 up to, but not including, March 1.

---

## 07 — Safe Practice: UPDATE and DELETE

The original seven seed rows should remain unchanged so that later sessions can use the same data.

Instead, create a temporary test row, update it, and then delete it.

### Step 1 — Insert a Test User

```sql
INSERT INTO users (full_name, email, created_at)
VALUES ('Test User', 'test.user@example.com', now());
```

### Step 2 — Update the Test User

```sql
UPDATE users
SET full_name = 'Updated Test User'
WHERE email = 'test.user@example.com';
```

### Step 3 — Verify the Updated Row

```sql
SELECT *
FROM users
WHERE email = 'test.user@example.com';
```

### Step 4 — Delete the Test User

```sql
DELETE FROM users
WHERE email = 'test.user@example.com';
```

### Step 5 — Verify the Original Data

```sql
SELECT COUNT(*) FROM users;
```

**Expected result:** 7 rows, assuming the table started with the original seven seed records and no other changes.

> **Safety reminder:** Always verify your `WHERE` condition before running `UPDATE` or `DELETE`.

---

## 08 — Key Takeaways

* CRUD stands for Create, Read, Update, and Delete.
* `INSERT`, `SELECT`, `UPDATE`, and `DELETE` implement CRUD in SQL.
* `WHERE` filters which rows a statement applies to.
* `AND`, `OR`, and `NOT` combine or negate conditions.
* `IN`, `BETWEEN`, `LIKE`, and `ILIKE` provide different filtering methods.
* `IS NULL` and `IS NOT NULL` check for missing values.
* A missing `WHERE` clause in an `UPDATE` or `DELETE` can affect every row.

---

## 🛠️ Technologies & Concepts

* PostgreSQL
* SQL
* CRUD Operations
* WHERE Clause
* Comparison Operators
* Logical Operators
* Pattern Matching
* Data Manipulation Language (DML)

---
