# Session 05 — Subqueries

**CodeBridge Japan — PostgreSQL Learning**

> Solve two-step questions by placing one SQL query inside another query.

---

## 📋 Session Overview

| Item             | Details                      |
| ---------------- | ---------------------------- |
| **Session**      | 05 / 08                      |
| **Module**       | Relationships & Real Queries |
| **Topic**        | Subqueries                   |
| **Prerequisite** | Session 04                   |
| **Level**        | Beginner → Intermediate      |
| **Database**     | PostgreSQL                   |

---

# 1. What Is a Subquery?

A **subquery** is a SQL query written inside another SQL query.

The query inside is called the **inner query**, and the query that contains it is called the **outer query**.

A subquery allows us to solve a problem in two logical steps:

1. Retrieve information using the inner query.
2. Use that information in the outer query.

### Example

```sql
SELECT name, price_cents
FROM products
WHERE price_cents > (
    SELECT AVG(price_cents)
    FROM products
);
```

The inner query calculates the average product price.

The outer query returns products whose prices are higher than that average.

---

# 2. Why Do We Need Subqueries?

Some business questions require us to find one piece of information before answering the main question.

For example:

> "Which products cost more than the average product price?"

We need to:

1. Calculate the average price.
2. Compare each product's price against that average.

A subquery combines these steps into a single SQL statement.

Subqueries are especially useful when checking whether a value belongs to a list or comparing a value against a calculated result.

---

# 3. Types of Subqueries

This session covers two common types.

| Type                | What It Returns               | Common Usage                                   |
| ------------------- | ----------------------------- | ---------------------------------------------- |
| **List Subquery**   | Multiple values in one column | `WHERE column IN (subquery)`                   |
| **Scalar Subquery** | A single value                | `WHERE column > (subquery)` or inside `SELECT` |

A scalar subquery must return one column and at most one row. If it returns multiple rows, PostgreSQL raises an error.

---

# 4. How a Subquery Works

Consider this query:

```sql
SELECT name, price_cents
FROM products
WHERE price_cents > (
    SELECT AVG(price_cents)
    FROM products
);
```

### Step 1 — Calculate the Average Price

```sql
SELECT AVG(price_cents)
FROM products;
```

Expected result:

```text
 avg
---------
 3799.2
```

### Step 2 — Compare Each Product Against the Average

The outer query effectively checks:

```text
price_cents > 3799.2
```

Only products with prices above the average are returned.

**Note:** This describes the logical relationship between the queries. PostgreSQL's optimizer may choose a different physical execution plan.

---

# 5. List Subquery — IN

## Find Users Who Have Placed Orders

The `IN` operator checks whether a value matches any value returned by a subquery.

### Syntax

```sql
SELECT columns
FROM table_a
WHERE column IN (
    SELECT matching_column
    FROM table_b
    WHERE condition
);
```

### Example

```sql
SELECT full_name
FROM users
WHERE id IN (
    SELECT DISTINCT user_id
    FROM orders
);
```

### Expected Result

```text
 full_name
----------------
 Tanvir Ahmed
 Mitsuki Sato
 Kenji Watanabe
 Aiko Tanaka
```

### Explanation

The inner query:

```sql
SELECT DISTINCT user_id
FROM orders;
```

returns the unique user IDs associated with orders.

```text
{1, 2, 4, 6}
```

The outer query finds the users whose IDs appear in that list.

`DISTINCT` removes duplicate IDs, so a user with multiple orders appears only once in the inner result.

---

# 6. Scalar Subquery — WHERE

A **scalar subquery** returns a single value.

We can use it with comparison operators such as:

```text
=   >   <   >=   <=   <>
```

## Find Products More Expensive Than the Average

```sql
SELECT
    name,
    price_cents
FROM products
WHERE price_cents > (
    SELECT AVG(price_cents)
    FROM products
);
```

### Expected Result

```text
 name                 | price_cents
----------------------+------------
 CodeBridge Hoodie    |        4500
 Mechanical Keyboard  |        8999
```

### Explanation

The inner query calculates the average price.

The outer query returns products whose prices are greater than that value.

This is a practical example of using a calculated value as a filter.

---

# 7. Scalar Subquery — SELECT

A scalar subquery can also appear inside the `SELECT` list.

This lets us display a calculated value alongside every row.

### Example

```sql
SELECT
    name,
    price_cents,
    (
        SELECT AVG(price_cents)
        FROM products
    ) AS avg_price
FROM products;
```

### Expected Result

```text
 name                          | price_cents | avg_price
-------------------------------+-------------+----------
 Beginner SQL Handbook         |        1999 |   3799.2
 PostgreSQL Cheat Sheet Poster |         999 |   3799.2
 CodeBridge Hoodie              |        4500 |   3799.2
 Mechanical Keyboard            |        8999 |   3799.2
 Desk Lamp                      |        2499 |   3799.2
```

The average price is displayed alongside each product.

---

# 8. NOT IN — Find Records Without Matches

The `NOT IN` operator checks whether a value does not appear in the list returned by a subquery.

## Find Products That Have Never Been Ordered

```sql
SELECT name
FROM products
WHERE id NOT IN (
    SELECT product_id
    FROM order_items
);
```

This returns products whose IDs do not appear in `order_items`.

### Important Note

`NOT IN` can behave unexpectedly if the subquery returns `NULL`.

If the subquery contains a `NULL`, the comparison may evaluate to unknown rather than true.

For this reason, `NOT EXISTS` is often a safer alternative when the subquery's values may contain `NULL`.

In our schema, `order_items.product_id` is declared `NOT NULL`, so this particular query avoids that issue.

---

# 9. More Subquery Examples

## Find the Most Expensive Product

```sql
SELECT name
FROM products
WHERE price_cents = (
    SELECT MAX(price_cents)
    FROM products
);
```

The inner query finds the maximum price.

The outer query returns the product or products with that price.

---

## Find Products Above a Specific Average

```sql
SELECT
    name,
    price_cents
FROM products
WHERE price_cents > (
    SELECT AVG(price_cents)
    FROM products
)
ORDER BY price_cents DESC;
```

This combines:

* Scalar subquery
* `AVG()`
* `WHERE`
* `ORDER BY`

---

# 10. Realistic Example — Loyalty Program

The marketing team wants to identify users who have placed at least one order.

These users will be eligible for a loyalty program.

We can solve this in two ways.

---

## Method 1 — JOIN + DISTINCT

```sql
SELECT DISTINCT
    u.full_name,
    u.email
FROM users AS u
INNER JOIN orders AS o
    ON o.user_id = u.id
ORDER BY u.full_name;
```

`DISTINCT` removes duplicate users who have placed multiple orders.

---

## Method 2 — Subquery + IN

```sql
SELECT
    full_name,
    email
FROM users
WHERE id IN (
    SELECT DISTINCT user_id
    FROM orders
)
ORDER BY full_name;
```

### Expected Result

```text
 full_name       | email
-----------------+-------------------------------
 Aiko Tanaka     | aiko.tanaka@example.com
 Kenji Watanabe  | kenji.watanabe@example.com
 Mitsuki Sato    | mitsuki.sato@example.com
 Tanvir Ahmed    | tanvir.ahmed@example.com
```

Both queries identify users who have placed at least one order.

---

# 11. JOIN vs Subquery

Both JOINs and subqueries can solve similar problems, but their structure and readability differ.

| Situation                                     | Common Approach                         |
| --------------------------------------------- | --------------------------------------- |
| Retrieve columns from multiple tables         | `JOIN`                                  |
| Check whether a value exists in another table | `IN` or `EXISTS`                        |
| Compare a value with an aggregate result      | Scalar subquery                         |
| Find records without matching records         | `NOT EXISTS` or `LEFT JOIN ... IS NULL` |

### Key Difference

**JOIN** is useful when we need to combine columns from related tables.

**Subquery** is useful when one query needs a result from another query, such as a list of IDs or a calculated value.

Neither approach is universally faster or better. The best choice depends on the problem and query structure.

---

# 12. Key Concepts Learned

* What a subquery is
* Inner query vs outer query
* List subqueries
* Scalar subqueries
* Using `IN` to match values against a list
* Using `NOT IN` to exclude matching values
* Using `AVG()` inside a subquery
* Using `MAX()` inside a subquery
* Scalar subqueries inside `SELECT`
* Finding products above the average price
* Finding products that have never been ordered
* Finding users who have placed orders
* Comparing JOINs with subqueries
* Understanding the `NULL` behavior of `NOT IN`

---

# 13. Key Takeaway

> **A subquery allows one SQL query to use the result of another query.**

Remember these common patterns:

### List Subquery

```sql
WHERE id IN (
    SELECT user_id
    FROM orders
);
```

### Scalar Subquery

```sql
WHERE price_cents > (
    SELECT AVG(price_cents)
    FROM products
);
```

### Exclusion Subquery

```sql
WHERE id NOT IN (
    SELECT product_id
    FROM order_items
);
```

These patterns are useful for filtering, comparisons, reporting, and checking relationships between tables.

---
