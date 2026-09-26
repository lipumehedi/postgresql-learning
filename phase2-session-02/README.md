# Session 02 — INNER JOIN

**CodeBridge Japan — PostgreSQL Learning**

> Combining rows from multiple tables by matching related columns.

---

## 📋 Session Overview

| Item             | Details                      |
| ---------------- | ---------------------------- |
| **Session**      | 02 / 08                      |
| **Module**       | Relationships & Real Queries |
| **Topic**        | INNER JOIN                   |
| **Prerequisite** | Session 01                   |
| **Level**        | Beginner → Intermediate      |
| **Database**     | PostgreSQL                   |

---

## 1. What Is a JOIN?

A **JOIN** combines rows from two or more tables by matching related columns.

In a relational database, information is intentionally separated into different tables to avoid duplication.

For example:

* `users` stores customer information.
* `orders` stores order information.
* `products` stores product information.
* `order_items` connects orders with products.

A JOIN allows us to bring this related information together when we need it.

### Basic Example

```sql
SELECT orders.id, users.full_name
FROM orders
INNER JOIN users
    ON orders.user_id = users.id;
```

### Expected Result

```text
 id | full_name
----+----------------
  1 | Tanvir Ahmed
  2 | Mitsuki Sato
  3 | Tanvir Ahmed
  4 | Kenji Watanabe
  5 | Aiko Tanaka
```

The `orders` table does not store the customer's name.

Instead:

```text
orders.user_id → users.id
```

The JOIN uses this relationship to retrieve the customer's name.

---

# 2. Why Do We Need JOINs?

In Session 01, the store database was split into multiple tables.

This avoids duplicated data and keeps the database organized.

However, real-world questions often require information from multiple tables.

For example:

> "Show me each order with the customer's name and the products they purchased."

That information may exist across:

```text
users
   ↓
orders
   ↓
order_items
   ↓
products
```

A JOIN allows us to combine these tables for a single query without storing the same information repeatedly.

---

# 3. How INNER JOIN Works

An `INNER JOIN` returns **only rows that have a matching row in both tables**.

### Example Relationship

```text
orders.user_id
      ↓
users.id
```

Suppose:

```text
orders
+----+---------+
| id | user_id |
+----+---------+
| 1  |    1    |
| 2  |    2    |
| 3  |    1    |
+----+---------+
```

And:

```text
users
+----+----------------+
| id | full_name      |
+----+----------------+
| 1  | Tanvir Ahmed   |
| 2  | Mitsuki Sato   |
| 3  | Farzana Rahman |
+----+----------------+
```

The JOIN matches:

```text
orders.user_id = users.id
```

Result:

```text
order_id | full_name
---------+----------------
1        | Tanvir Ahmed
2        | Mitsuki Sato
3        | Tanvir Ahmed
```

`Farzana Rahman` does not appear because she has no matching order.

### Key Rule

> **INNER JOIN keeps only matching rows.**

Users without orders are excluded from the result.

This becomes important when comparing `INNER JOIN` with `LEFT JOIN`, which will be covered in Session 03.

---

# 4. INNER JOIN Syntax

## Basic Two-Table JOIN

```sql
SELECT columns
FROM table_a
INNER JOIN table_b
    ON table_a.foreign_key = table_b.id;
```

The most important part is the `ON` condition.

It tells PostgreSQL **how the two tables are related**.

---

## Find the Customer for Order 1

```sql
SELECT
    orders.id AS order_id,
    users.full_name
FROM orders
INNER JOIN users
    ON orders.user_id = users.id
WHERE orders.id = 1;
```

### Expected Result

```text
 order_id | full_name
----------+---------------
        1 | Tanvir Ahmed
```

---

# 5. Table Aliases

When queries become larger, writing full table names repeatedly can make SQL harder to read.

We can use **aliases**.

```sql
SELECT
    o.id AS order_id,
    u.full_name
FROM orders AS o
INNER JOIN users AS u
    ON o.user_id = u.id;
```

Here:

```text
o → orders
u → users
```

The `AS` keyword is optional.

Both are valid:

```sql
FROM orders AS o
```

and:

```sql
FROM orders o
```

Aliases are especially useful when working with multiple JOINs.

---

# 6. Joining Three or More Tables

JOINs can be chained together.

### General Pattern

```sql
SELECT columns
FROM table_a
INNER JOIN table_b
    ON table_a.b_id = table_b.id
INNER JOIN table_c
    ON table_b.c_id = table_c.id;
```

For example, to find the products included in order `1`:

```sql
SELECT
    oi.order_id,
    p.name,
    oi.quantity
FROM order_items AS oi
INNER JOIN products AS p
    ON oi.product_id = p.id
WHERE oi.order_id = 1;
```

### Expected Result

```text
 order_id | name                          | quantity
----------+-------------------------------+---------
        1 | Beginner SQL Handbook        |       1
        1 | PostgreSQL Cheat Sheet Poster|       2
```

The relationship is:

```text
order_items.product_id
          ↓
products.id
```

---

# 7. Basic JOIN Examples

## Products with Category Names

```sql
SELECT
    p.name,
    c.name AS category
FROM products AS p
INNER JOIN categories AS c
    ON p.category_id = c.id;
```

This combines:

```text
products
    ↓
categories
```

---

## Payments with Order Status

```sql
SELECT
    pay.id AS payment_id,
    o.status
FROM payments AS pay
INNER JOIN orders AS o
    ON pay.order_id = o.id;
```

This combines:

```text
payments
    ↓
orders
```

Now we can see the status of the order associated with each payment.

---

# 8. Realistic Example — Full Order Receipt

A real store often needs more than just the customer's name.

For example:

> Show the customer, products purchased, quantity, unit price, and total price for an order.

This requires four tables:

```text
users
  ↓
orders
  ↓
order_items
  ↓
products
```

### Query

```sql
SELECT
    u.full_name,
    p.name AS product,
    oi.quantity,
    ROUND(oi.unit_price_cents / 100.0, 2) AS unit_price,
    ROUND(
        (oi.unit_price_cents * oi.quantity) / 100.0,
        2
    ) AS line_total
FROM orders AS o
INNER JOIN users AS u
    ON o.user_id = u.id
INNER JOIN order_items AS oi
    ON oi.order_id = o.id
INNER JOIN products AS p
    ON oi.product_id = p.id
WHERE o.id = 1;
```

### Expected Result

```text
 full_name    | product                        | quantity | unit_price | line_total
--------------+--------------------------------+----------+------------+-----------
 Tanvir Ahmed | Beginner SQL Handbook          |        1 |      19.99 |      19.99
 Tanvir Ahmed | PostgreSQL Cheat Sheet Poster  |        2 |       9.99 |      19.98
```

---

# 9. JOIN + Other SQL Features

A JOIN does not replace other SQL features.

The same SQL tools from Phase 1 can still be used:

### WHERE

```sql
WHERE o.id = 1
```

### Aliases

```sql
p.name AS product
```

### Functions

```sql
ROUND(oi.unit_price_cents / 100.0, 2)
```

### Conditions

```sql
WHERE o.status = 'completed'
```

JOIN simply determines **which related rows are available to the query**.

---

# 10. INNER JOIN vs No Matching Row

One of the most important concepts to remember:

```text
INNER JOIN
    ↓
Only matching rows
```

For example, if a user has never placed an order:

```text
users
id = 3
name = Farzana Rahman
```

but there is no:

```text
orders.user_id = 3
```

then Farzana will **not** appear in:

```sql
SELECT
    u.full_name,
    o.id AS order_id
FROM users AS u
INNER JOIN orders AS o
    ON u.id = o.user_id;
```

This is expected behavior.

To include users even when they have no orders, we will need:

```sql
LEFT JOIN
```

which is the focus of **Session 03**.

---

# 11. JOIN Relationship Map

The CodeBridge store schema can now be queried through its relationships:

```text
users
  │
  │ user_id
  ▼
orders
  │
  │ order_id
  ▼
order_items
  │
  │ product_id
  ▼
products
  │
  │ category_id
  ▼
categories
```

Payments are connected directly to orders:

```text
orders
  │
  │ order_id
  ▼
payments
```

These foreign-key relationships are what make multi-table queries possible.

---

# 12. Key Concepts Learned

* What a `JOIN` does
* What an `INNER JOIN` does
* Matching rows using `ON`
* Foreign key → primary key relationships
* Joining two tables
* Joining three or more tables
* Table aliases
* Filtering joined results with `WHERE`
* Using SQL functions inside JOIN queries
* Understanding unmatched rows
* Building realistic multi-table reports

---

# 13. Key Takeaway

> **INNER JOIN combines related rows from multiple tables and returns only the rows that have matching records on both sides.**

The most important pattern to remember is:

```sql
SELECT columns
FROM table_a
INNER JOIN table_b
    ON table_a.foreign_key = table_b.id;
```

And for larger queries:

```sql
SELECT columns
FROM table_a
INNER JOIN table_b
    ON table_a.foreign_key = table_b.id
INNER JOIN table_c
    ON table_b.foreign_key = table_c.id;
```
