# Session 07 — Aggregation

> **Module:** Answering Questions
> **Sheet:** 07 / 08
> **Level:** Absolute Beginner
> **Prerequisite:** Session 06 — Sorting & Pagination Basics
> **Database:** PostgreSQL

---

## 🎯 Learning Objectives

By the end of this session, I will be able to:

* Understand aggregate functions.
* Count rows using `COUNT()`.
* Calculate totals using `SUM()`.
* Calculate averages using `AVG()`.
* Find minimum and maximum values using `MIN()` and `MAX()`.
* Group rows using `GROUP BY`.
* Filter groups using `HAVING`.
* Combine `WHERE`, `GROUP BY`, and `HAVING` in a single query.
* Generate basic business reports using SQL.

---

## 01 — What Is Aggregation?

### Turning Many Rows into One Answer

An **aggregate function** takes multiple rows and produces a single summarized value.

PostgreSQL provides five commonly used aggregate functions:

| Function  | Purpose                                 |
| --------- | --------------------------------------- |
| `COUNT()` | Count rows or non-NULL values           |
| `SUM()`   | Calculate the total of numeric values   |
| `AVG()`   | Calculate the average of numeric values |
| `MIN()`   | Find the smallest value                 |
| `MAX()`   | Find the largest value                  |

### Example

```sql
-- Count all rows
SELECT COUNT(*) FROM users;

-- Find the earliest signup
SELECT MIN(created_at) FROM users;

-- Find the latest signup
SELECT MAX(created_at) FROM users;
```

### What Is GROUP BY?

`GROUP BY` divides rows into groups based on one or more columns. Aggregate functions then calculate a result for each group.

For example:

* Total number of users → One result.
* Number of users per month → One result per month.

### What Is HAVING?

`HAVING` filters groups after aggregation.

For example, you can display only the months that had more than two signups.

---

## 02 — Why Do We Need Aggregation?

Many business questions require summarized data rather than individual records.

Examples:

* How many students are registered?
* When did the first student join?
* Which month had the most signups?
* What is the total revenue?
* What is the average product price?

Aggregate functions help answer these questions directly in PostgreSQL.

### WHERE vs HAVING

| Clause   | Purpose                   | Applied         |
| -------- | ------------------------- | --------------- |
| `WHERE`  | Filters individual rows   | Before grouping |
| `HAVING` | Filters aggregated groups | After grouping  |

`WHERE` cannot directly filter a group's aggregate result because that result has not been calculated yet.

---

## 03 — How Aggregation Works

### Group → Aggregate → Filter

A grouped query follows this logical sequence:

1. `WHERE` filters individual rows.
2. `GROUP BY` organizes the remaining rows into groups.
3. Aggregate functions calculate values for each group.
4. `HAVING` filters groups based on aggregate results.
5. `ORDER BY` sorts the resulting groups.
6. `LIMIT` and `OFFSET` can restrict the final result.

### Example: Count Signups by Month

The `users` table contains seven users:

| Month    | Users                                        |
| -------- | -------------------------------------------- |
| January  | Tanvir Ahmed, Mitsuki Sato                   |
| February | Farzana Rahman, Kenji Watanabe, Rashed Karim |
| March    | Aiko Tanaka, Nusrat Jahan                    |

After grouping and counting:

| Month    | COUNT(*) |
| -------- | -------: |
| January  |        2 |
| February |        3 |
| March    |        2 |

If we use `HAVING COUNT(*) > 2`, only February remains.

---

## 04 — SQL Syntax

### 4.1 COUNT()

Counts rows or non-NULL values.

```sql
-- Count all rows
SELECT COUNT(*)
FROM table_name;

-- Count non-NULL values in a column
SELECT COUNT(column_name)
FROM table_name;
```

### 4.2 SUM()

Calculates the total of numeric values.

```sql
SELECT SUM(column_name)
FROM table_name;
```

### 4.3 AVG()

Calculates the average of numeric values.

```sql
SELECT AVG(column_name)
FROM table_name;
```

### 4.4 MIN() and MAX()

Find the smallest and largest values.

```sql
SELECT MIN(column_name)
FROM table_name;

SELECT MAX(column_name)
FROM table_name;
```

### 4.5 GROUP BY

Groups rows based on a column.

```sql
SELECT group_column, COUNT(*)
FROM table_name
GROUP BY group_column;
```

### 4.6 HAVING

Filters groups based on an aggregate condition.

```sql
SELECT group_column, COUNT(*)
FROM table_name
GROUP BY group_column
HAVING COUNT(*) > 2;
```

---

## 05 — Basic Examples Using the Users Table

### Count All Users

```sql
SELECT COUNT(*)
FROM users;
```

**Expected result:** `7`

### Find the Earliest Signup

```sql
SELECT MIN(created_at)
FROM users;
```

**Expected result:** `2024-01-15 09:12:00+09`

### Find the Latest Signup

```sql
SELECT MAX(created_at)
FROM users;
```

**Expected result:** `2024-03-05 16:40:00+09`

### Calculate the Sum of IDs

```sql
SELECT SUM(id)
FROM users;
```

**Expected result:** `28`

### Calculate the Average ID

```sql
SELECT AVG(id)
FROM users;
```

**Expected result:** `4.0000000000000000`

> **Note:** `id` is used here only to demonstrate `SUM()` and `AVG()`. An ID is not a meaningful business measurement. A real numeric business column will be introduced in Session 08.

---

## 06 — GROUP BY: Count Users per Month

To group users by signup month, PostgreSQL's `date_trunc()` function can round each timestamp down to the beginning of its month.

```sql
SELECT
    date_trunc('month', created_at) AS signup_month,
    COUNT(*) AS total_signups
FROM users
GROUP BY signup_month
ORDER BY signup_month;
```

### Expected Result

| Signup Month | Total Signups |
| ------------ | ------------: |
| 2024-01-01   |             2 |
| 2024-02-01   |             3 |
| 2024-03-01   |             2 |

---

## 07 — HAVING: Filter Groups

Suppose CodeBridge staff want to find months with more than two signups.

```sql
SELECT
    date_trunc('month', created_at) AS signup_month,
    COUNT(*) AS total_signups
FROM users
GROUP BY signup_month
HAVING COUNT(*) > 2
ORDER BY signup_month;
```

### Expected Result

| Signup Month | Total Signups |
| ------------ | ------------: |
| 2024-02-01   |             3 |

Only February meets the condition.

### Important GROUP BY Rule

Every selected column in a grouped query must either:

* Be included in the `GROUP BY` clause, or
* Be used inside an aggregate function.

Otherwise, PostgreSQL generally raises a grouping error.

---

## 08 — Realistic Example: Staff Signup Report

### Scenario

CodeBridge ran a recruiting campaign in February 2024 and wants to know how many students joined during that period.

### Count February Signups

```sql
SELECT COUNT(*) AS campaign_signups
FROM users
WHERE created_at >= '2024-02-01'
  AND created_at < '2024-03-01';
```

### Expected Result

| Campaign Signups |
| ---------------: |
|                3 |

### Combine WHERE, GROUP BY, and HAVING

Now suppose the staff want a monthly report showing months with at least two signups from 2024 onward.

```sql
SELECT
    date_trunc('month', created_at) AS signup_month,
    COUNT(*) AS total
FROM users
WHERE created_at >= '2024-01-01'
GROUP BY signup_month
HAVING COUNT(*) >= 2
ORDER BY signup_month;
```

### Expected Result

| Signup Month | Total |
| ------------ | ----: |
| 2024-01-01   |     2 |
| 2024-02-01   |     3 |
| 2024-03-01   |     2 |

This query combines three important concepts:

* `WHERE` filters rows before grouping.
* `GROUP BY` groups rows by month.
* `HAVING` filters groups based on the count.

---

---

## 🧠 Key Takeaways

* Aggregate functions summarize multiple rows.
* `COUNT()`, `SUM()`, `AVG()`, `MIN()`, and `MAX()` are commonly used aggregate functions.
* `GROUP BY` calculates summaries for individual groups.
* `HAVING` filters groups after aggregation.
* `WHERE` filters individual rows before aggregation.
* `date_trunc()` can group timestamps by month.
* Grouped queries require selected columns to be grouped or aggregated.

---

## 🛠️ Technologies & Concepts

* PostgreSQL
* SQL
* Aggregate Functions
* `COUNT()`
* `SUM()`
* `AVG()`
* `MIN()` and `MAX()`
* `GROUP BY`
* `HAVING`
* `WHERE`
* `date_trunc()`

---
