# Session 06 — Sorting & Pagination Basics

> **Module:** Working with Data
> **Sheet:** 06 / 08
> **Level:** Absolute Beginner
> **Prerequisite:** Session 05 — CRUD & WHERE
> **Database:** PostgreSQL

---

## 🎯 Learning Objectives

By the end of this session, I will be able to:

* Sort query results using `ORDER BY`.
* Understand ascending (`ASC`) and descending (`DESC`) order.
* Limit the number of rows using `LIMIT`.
* Skip rows using `OFFSET`.
* Build basic pagination queries.
* Sort data using multiple columns.
* Understand how PostgreSQL sorts and slices query results.

---

## 01 — What Is Sorting & Pagination?

### Ordering Results with ORDER BY

By default, PostgreSQL does not guarantee any particular order for rows returned by a `SELECT` query.

The `ORDER BY` clause specifies the order in which rows should appear.

```sql
-- A to Z
SELECT full_name
FROM users
ORDER BY full_name ASC;

-- Z to A
SELECT full_name
FROM users
ORDER BY full_name DESC;
```

| Keyword | Meaning                               |
| ------- | ------------------------------------- |
| `ASC`   | Ascending order: smallest to largest  |
| `DESC`  | Descending order: largest to smallest |

If neither is specified, PostgreSQL uses `ASC`.

### Limiting and Skipping Rows

`LIMIT` and `OFFSET` allow us to retrieve a smaller portion of a query result.

```sql
-- Return the first 3 rows
SELECT full_name
FROM users
ORDER BY full_name ASC
LIMIT 3;

-- Skip the first 3 rows and return the next 3
SELECT full_name
FROM users
ORDER BY full_name ASC
LIMIT 3 OFFSET 3;
```

* `LIMIT 3` returns at most three rows.
* `OFFSET 3` skips the first three rows before returning results.

Together, they form the foundation of pagination in web applications.

---

## 02 — Why Do We Need Sorting & Pagination?

### Rows Have No Guaranteed Order

Think of a database table as a box of index cards, not a permanently sorted stack.

Even if records were inserted in a particular order, PostgreSQL does not guarantee that a query will return them in that same order.

For example, if an application needs to display the newest users first, it should explicitly use:

```sql
SELECT *
FROM users
ORDER BY created_at DESC;
```

### Why Pagination Matters

A database may contain millions of records, but a webpage usually displays only a limited number at a time.

Pagination helps an application:

* Display a manageable number of records per page.
* Avoid retrieving unnecessary rows.
* Build features such as page navigation and load-more buttons.

---

## 03 — How Does It Work?

### Sort First, Then Slice

A query that uses filtering, sorting, and pagination follows this logical sequence:

1. `WHERE` — Filter the matching rows.
2. `ORDER BY` — Sort the remaining rows.
3. `OFFSET` — Skip the specified number of rows.
4. `LIMIT` — Return the specified number of rows.

### Example: Newest Users First

```sql
SELECT full_name, created_at
FROM users
ORDER BY created_at DESC
LIMIT 3 OFFSET 2;
```

The query sorts all matching rows by date, newest first. It then skips the first two rows and returns the next three.

| Position | Full Name      | Date       |
| -------: | -------------- | ---------- |
|        1 | Nusrat Jahan   | 2024-03-05 |
|        2 | Aiko Tanaka    | 2024-03-01 |
|        3 | Rashed Karim   | 2024-02-14 |
|        4 | Kenji Watanabe | 2024-02-10 |
|        5 | Farzana Rahman | 2024-02-02 |
|        6 | Mitsuki Sato   | 2024-01-18 |
|        7 | Tanvir Ahmed   | 2024-01-15 |

With `OFFSET 2`, PostgreSQL skips Nusrat Jahan and Aiko Tanaka, then returns:

* Rashed Karim
* Kenji Watanabe
* Farzana Rahman

**Key concept:** Sorting happens before pagination.

---

## 04 — SQL Syntax

### ORDER BY — Ascending Order

```sql
SELECT *
FROM table_name
ORDER BY column_name ASC;
```

### ORDER BY — Descending Order

```sql
SELECT *
FROM table_name
ORDER BY column_name DESC;
```

### LIMIT — Return a Fixed Number of Rows

```sql
SELECT *
FROM table_name
ORDER BY column_name
LIMIT n;
```

### LIMIT + OFFSET — Skip and Return Rows

```sql
SELECT *
FROM table_name
ORDER BY column_name
LIMIT n OFFSET m;
```

Here:

* `n` = Maximum number of rows to return.
* `m` = Number of rows to skip.

### Sorting by Multiple Columns

You can sort by more than one column. Later columns determine the order when earlier columns have equal values.

```sql
SELECT *
FROM table_name
ORDER BY column_a ASC, column_b DESC;
```

---

## 05 — Basic Examples: Sorting the Users Table

### Oldest Signups First

```sql
SELECT full_name, created_at
FROM users
ORDER BY created_at ASC;
```

### Newest Signups First

```sql
SELECT full_name, created_at
FROM users
ORDER BY created_at DESC;
```

### Return the Three Newest Users

```sql
SELECT full_name, created_at
FROM users
ORDER BY created_at DESC
LIMIT 3;
```

### Expected Result

| Full Name    | Created At             |
| ------------ | ---------------------- |
| Nusrat Jahan | 2024-03-05 16:40:00+09 |
| Aiko Tanaka  | 2024-03-01 10:00:00+09 |
| Rashed Karim | 2024-02-14 19:22:00+09 |

---

## 06 — Realistic Example: Pagination

### Scenario

CodeBridge's admin dashboard displays three students per page, with the newest signups first.

### Page 1

```sql
SELECT full_name, created_at
FROM users
ORDER BY created_at DESC
LIMIT 3 OFFSET 0;
```

### Page 2

```sql
SELECT full_name, created_at
FROM users
ORDER BY created_at DESC
LIMIT 3 OFFSET 3;
```

### Expected Result — Page 2

| Full Name      | Created At             |
| -------------- | ---------------------- |
| Kenji Watanabe | 2024-02-10 11:47:00+09 |
| Farzana Rahman | 2024-02-02 08:05:00+09 |
| Mitsuki Sato   | 2024-01-18 14:30:00+09 |

### Pagination Formula

The `OFFSET` value depends on the page number and page size.

$$
\text{OFFSET} = (\text{page number} - 1) \times \text{page size}
$$

For a page size of 3:

| Page | LIMIT | OFFSET |
| ---: | ----: | -----: |
|    1 |     3 |      0 |
|    2 |     3 |      3 |
|    3 |     3 |      6 |

### Important Performance Note

On very large tables, high `OFFSET` values can become inefficient because PostgreSQL still has to process the rows being skipped.

For large datasets, cursor-based or keyset pagination can be a more efficient alternative.

---

---

## 07 — Practice Exercises

Try writing the SQL queries yourself.

* [ ] Display all users alphabetically by name.
* [ ] Display all users in reverse alphabetical order.
* [ ] Find the three oldest signups.
* [ ] Find the two newest signups.
* [ ] Display page 1 with two users per page.
* [ ] Display page 2 with two users per page.
* [ ] Sort users by `created_at` descending and `id` descending.

---

## 🧠 Key Takeaways

* `ORDER BY` controls the order of query results.
* `ASC` means ascending; `DESC` means descending.
* `LIMIT` controls the maximum number of returned rows.
* `OFFSET` skips rows before returning results.
* Pagination uses `LIMIT` and `OFFSET` together.
* Sorting should be explicit whenever the order matters.
* A unique tie-breaker helps make pagination more consistent.

---

## 🛠️ Technologies & Concepts

* PostgreSQL
* SQL
* `ORDER BY`
* `ASC` and `DESC`
* `LIMIT`
* `OFFSET`
* Pagination
* Query Performance

---
