# Phase 3 – Session 02: Window Functions I (ROW_NUMBER, RANK & PARTITION BY)

**CodeBridge Japan – PostgreSQL Learning Journey**

## Session Overview

| Item         | Details                                                      |
| ------------ | ------------------------------------------------------------ |
| Phase        | Phase 3 – Advanced Queries & Real-World Patterns             |
| Session      | 02 / 08                                                      |
| Topic        | Window Functions I                                           |
| Concepts     | `ROW_NUMBER()`, `RANK()`, `PARTITION BY`, `ORDER BY`, `OVER` |
| Prerequisite | Session 01 – Common Table Expressions (CTEs)                 |
| Level        | Beginner to Intermediate                                     |

---

## 1. Introduction to Window Functions

A **window function** performs calculations across a set of related rows without combining them into a single row.

Unlike `GROUP BY`, which aggregates rows, window functions preserve individual rows and add a calculated value to each row.

### Window Functions vs. GROUP BY

| Feature                   | GROUP BY    | Window Functions                 |
| ------------------------- | ----------- | -------------------------------- |
| Groups rows               | Yes         | Logically partitions rows        |
| Preserves individual rows | No          | Yes                              |
| Returns one row per group | Usually     | Returns one result per input row |
| Typical use               | Aggregation | Ranking, numbering, analytics    |

### Two Important Window Functions

| Function       | Description                                                           |
| -------------- | --------------------------------------------------------------------- |
| `ROW_NUMBER()` | Assigns a unique sequential number to each row within a window.       |
| `RANK()`       | Assigns the same rank to tied rows and skips subsequent rank numbers. |

### Basic Syntax

```sql
ROW_NUMBER() OVER (
    PARTITION BY group_column
    ORDER BY sort_column
)
```

The `OVER` clause defines the window used by the function.

---

## 2. Why Window Functions Are Useful

Suppose a business wants to identify customers who have placed multiple orders.

Using `GROUP BY` and `COUNT()` can show the number of orders per customer, but it does not preserve the details of each individual order.

Window functions allow us to:

* Number each customer's orders in chronological order.
* Identify first-time and repeat orders.
* Rank products within each category.
* Analyze individual records while retaining group-level context.

---

## 3. Understanding PARTITION BY and ORDER BY

### PARTITION BY

`PARTITION BY` divides rows into separate groups for the window function.

Each group is processed independently, and numbering or ranking starts again within each partition.

### ORDER BY

`ORDER BY` inside the `OVER` clause determines the order in which rows are numbered or ranked.

### Example: Number Orders Per Customer

```sql
SELECT
    order_id,
    user_id,
    created_at,
    ROW_NUMBER() OVER (
        PARTITION BY user_id
        ORDER BY created_at
    ) AS order_number
FROM orders;
```

### How It Works

* `PARTITION BY user_id` creates a separate window for each customer.
* `ORDER BY created_at` sorts each customer's orders by creation time.
* `ROW_NUMBER()` assigns sequential numbers starting at 1 for each customer.

For example:

| user_id | order_id | created_at | order_number |
| ------- | -------- | ---------- | ------------ |
| 1       | 1        | 2024-04-01 | 1            |
| 1       | 3        | 2024-04-10 | 2            |
| 2       | 2        | 2024-04-03 | 1            |
| 3       | 4        | 2024-04-12 | 1            |

Each customer's numbering starts from 1 independently.

---

## 4. ROW_NUMBER()

`ROW_NUMBER()` assigns a unique sequential number to every row within a window.

### Syntax

```sql
SELECT
    columns,
    ROW_NUMBER() OVER (
        PARTITION BY group_column
        ORDER BY sort_column
    ) AS row_num
FROM table_name;
```

### Example: Number Each Customer's Orders

```sql
SELECT
    o.id AS order_id,
    u.full_name,
    o.created_at,
    ROW_NUMBER() OVER (
        PARTITION BY o.user_id
        ORDER BY o.created_at
    ) AS order_number
FROM orders AS o
JOIN users AS u
    ON u.id = o.user_id
ORDER BY
    u.full_name,
    o.created_at;
```

### Expected Output

| order_id | full_name      | created_at | order_number |
| -------- | -------------- | ---------- | ------------ |
| 5        | Aiko Tanaka    | 2024-04-15 | 1            |
| 4        | Kenji Watanabe | 2024-04-12 | 1            |
| 2        | Mitsuki Sato   | 2024-04-03 | 1            |
| 1        | Tanvir Ahmed   | 2024-04-01 | 1            |
| 3        | Tanvir Ahmed   | 2024-04-10 | 2            |

**Key observation:** Tanvir Ahmed's second order receives `order_number = 2`, indicating that it is not his first order.

---

## 5. RANK()

`RANK()` assigns ranking numbers based on the specified ordering.

When multiple rows have the same ordering value, they receive the same rank. The next rank is skipped accordingly.

### Syntax

```sql
SELECT
    columns,
    RANK() OVER (
        PARTITION BY group_column
        ORDER BY sort_column DESC
    ) AS rank_num
FROM table_name;
```

### Example: Compare ROW_NUMBER() and RANK()

```sql
SELECT
    name,
    price,
    ROW_NUMBER() OVER (
        ORDER BY price DESC
    ) AS row_num,
    RANK() OVER (
        ORDER BY price DESC
    ) AS rank_num
FROM (
    VALUES
        ('Item A', 100),
        ('Item B', 100),
        ('Item C', 90)
) AS t(name, price);
```

### Expected Output

| name   | price | row_num | rank_num |
| ------ | ----: | ------: | -------: |
| Item A |   100 |       1 |        1 |
| Item B |   100 |       2 |        1 |
| Item C |    90 |       3 |        3 |

### Key Difference

* `ROW_NUMBER()` assigns a unique number to each row, even when values are tied.
* `RANK()` assigns the same rank to tied rows and skips the next rank.

For the values above, `RANK()` returns `1, 1, 3`, while `ROW_NUMBER()` returns `1, 2, 3`.

---

## 6. Practical Example: Rank Products Within Each Category

Use `RANK()` to rank products by price within their respective categories.

```sql
SELECT
    c.name AS category,
    p.name AS product,
    p.price_cents,
    RANK() OVER (
        PARTITION BY p.category_id
        ORDER BY p.price_cents DESC
    ) AS price_rank
FROM products AS p
JOIN categories AS c
    ON c.id = p.category_id
ORDER BY
    category,
    price_rank;
```

### Expected Output

| category   | product                       | price_cents | price_rank |
| ---------- | ----------------------------- | ----------: | ---------: |
| Books      | Beginner SQL Handbook         |        1999 |          1 |
| Gear       | Mechanical Keyboard           |        8999 |          1 |
| Gear       | CodeBridge Hoodie             |        4500 |          2 |
| Gear       | Desk Lamp                     |        2499 |          3 |
| Study Aids | PostgreSQL Cheat Sheet Poster |         999 |          1 |

**Key observation:** Each category has its own ranking. The highest-priced product in each category receives rank 1.

---

## 7. Real-World Example: Identify Repeat Orders

### Business Scenario

The marketing team wants to identify orders that are not a customer's first order.

We can use `ROW_NUMBER()` to number each customer's orders and then filter for rows where `order_number > 1`.

### Incorrect Approach

```sql
SELECT
    o.id AS order_id,
    u.full_name,
    ROW_NUMBER() OVER (
        PARTITION BY o.user_id
        ORDER BY o.created_at
    ) AS order_number
FROM orders AS o
JOIN users AS u
    ON u.id = o.user_id
WHERE order_number > 1;
```

### Why Does This Fail?

The query produces an error because the window function's alias `order_number` is not available to the `WHERE` clause at that stage of query processing.

### Correct Approach: Use a CTE

```sql
WITH numbered_orders AS (
    SELECT
        o.id AS order_id,
        u.full_name,
        ROW_NUMBER() OVER (
            PARTITION BY o.user_id
            ORDER BY o.created_at
        ) AS order_number
    FROM orders AS o
    JOIN users AS u
        ON u.id = o.user_id
)
SELECT *
FROM numbered_orders
WHERE order_number > 1;
```

### Expected Output

| order_id | full_name    | order_number |
| -------- | ------------ | -----------: |
| 3        | Tanvir Ahmed |            2 |

### Key Takeaway

The CTE calculates the window function first. The outer query then filters the calculated result.

This pattern is useful whenever you need to filter rows based on a window function's output.

---

## 8. Key Concepts Learned

* Window functions preserve individual rows while calculating values across related rows.
* `OVER` defines the window for a window function.
* `PARTITION BY` separates rows into independent groups.
* `ORDER BY` controls the numbering or ranking sequence.
* `ROW_NUMBER()` assigns unique sequential numbers.
* `RANK()` assigns equal ranks to tied values and skips subsequent ranks.
* Window function results cannot be directly filtered in the same query's `WHERE` clause.
* CTEs can be used to calculate window functions first and filter the results in an outer query.

---

## 9. Session Summary

| Concept               | Purpose                                           |
| --------------------- | ------------------------------------------------- |
| `OVER`                | Defines the window                                |
| `PARTITION BY`        | Divides rows into groups                          |
| `ORDER BY`            | Determines the order within each window           |
| `ROW_NUMBER()`        | Assigns sequential row numbers                    |
| `RANK()`              | Assigns ranks, handling ties                      |
| CTE + Window Function | Filters results based on calculated window values |

### Key Takeaway

**Window functions let you analyze individual rows without losing the details that `GROUP BY` would aggregate away.**

---