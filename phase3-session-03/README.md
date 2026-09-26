# Session 03 — LEFT JOIN, RIGHT JOIN & FULL JOIN

**CodeBridge Japan — PostgreSQL Learning**

> Keeping unmatched rows in the result — going beyond the limitations of `INNER JOIN`.

---

## 📋 Session Overview

| Item             | Details                           |
| ---------------- | --------------------------------- |
| **Session**      | 03 / 08                           |
| **Module**       | Relationships & Real Queries      |
| **Topic**        | LEFT JOIN, RIGHT JOIN & FULL JOIN |
| **Prerequisite** | Session 02                        |
| **Level**        | Beginner → Intermediate           |
| **Database**     | PostgreSQL                        |

---

# 1. Why Do We Need OUTER JOINs?

In Session 02, we learned that `INNER JOIN` returns only rows that have a match in both tables.

But sometimes we need to keep rows even when there is **no matching record**.

For example:

> "Show every user, along with their order if they have one."

With `INNER JOIN`, users who have never placed an order disappear from the result.

`LEFT JOIN`, `RIGHT JOIN`, and `FULL JOIN` solve this problem by keeping unmatched rows and filling the missing columns with `NULL`.

These are commonly called **OUTER JOINs**.

---

# 2. LEFT JOIN

## Keep Every Row from the Left Table

The basic structure is:

```sql
SELECT columns
FROM table_a
LEFT JOIN table_b
    ON table_a.foreign_key = table_b.id;
```

A `LEFT JOIN` means:

> **Keep every row from the left table, whether or not a matching row exists in the right table.**

---

## Example: Every User and Their Orders

```sql
SELECT
    u.full_name,
    o.id AS order_id,
    o.status
FROM users AS u
LEFT JOIN orders AS o
    ON o.user_id = u.id
ORDER BY u.id;
```

### Expected Result

```text
 full_name        | order_id | status
------------------+----------+-----------
 Tanvir Ahmed     |        1 | completed
 Tanvir Ahmed     |        3 | pending
 Mitsuki Sato     |        2 | completed
 Farzana Rahman   |     NULL | NULL
 Kenji Watanabe   |        4 | completed
 Rashed Karim     |     NULL | NULL
 Aiko Tanaka      |        5 | cancelled
 Nusrat Jahan     |     NULL | NULL
```

Users such as:

* Farzana Rahman
* Rashed Karim
* Nusrat Jahan

are still included even though they have no orders.

Their order columns contain `NULL`.

---

# 3. LEFT JOIN + IS NULL

One of the most useful SQL patterns is:

```text
LEFT JOIN + IS NULL
```

This allows us to find records that **do not have a matching record**.

### Find Users Without Orders

```sql
SELECT
    u.full_name
FROM users AS u
LEFT JOIN orders AS o
    ON o.user_id = u.id
WHERE o.id IS NULL;
```

### Expected Result

```text
 full_name
----------------
 Farzana Rahman
 Rashed Karim
 Nusrat Jahan
```

The logic is:

```text
users
   ↓
LEFT JOIN orders
   ↓
No matching order
   ↓
order columns become NULL
   ↓
WHERE o.id IS NULL
   ↓
Users without orders
```

### Key Pattern

> **"Find everything that has no matching record" → `LEFT JOIN` + `IS NULL`**

This pattern is extremely useful in real-world applications.

---

# 4. LEFT JOIN vs INNER JOIN

Consider this relationship:

```text
users
+----+----------------+
| id | full_name      |
+----+----------------+
| 1  | Tanvir Ahmed   |
| 2  | Mitsuki Sato   |
| 3  | Farzana Rahman |
+----+----------------+

orders
+----+---------+
| id | user_id |
+----+---------+
| 1  |    1    |
| 2  |    2    |
+----+---------+
```

### INNER JOIN

```text
Tanvir Ahmed
Mitsuki Sato
```

Only matching users appear.

### LEFT JOIN

```text
Tanvir Ahmed
Mitsuki Sato
Farzana Rahman
```

Farzana is included because the `users` table is on the left.

Her order columns become:

```text
NULL
```

---

# 5. RIGHT JOIN

## Keep Every Row from the Right Table

`RIGHT JOIN` is the opposite of `LEFT JOIN`.

```sql
SELECT columns
FROM table_a
RIGHT JOIN table_b
    ON table_a.foreign_key = table_b.id;
```

It means:

> **Keep every row from the right table, whether or not a matching row exists on the left.**

---

## Example

```sql
SELECT
    u.full_name,
    o.id AS order_id
FROM orders AS o
RIGHT JOIN users AS u
    ON o.user_id = u.id;
```

This keeps every row from `users`.

In other words, it produces the same logical result as:

```sql
SELECT
    u.full_name,
    o.id AS order_id
FROM users AS u
LEFT JOIN orders AS o
    ON o.user_id = u.id;
```

### Why Is RIGHT JOIN Used Less Often?

`RIGHT JOIN` is essentially the mirror image of `LEFT JOIN`.

Instead of:

```sql
orders
RIGHT JOIN users
```

we can simply write:

```sql
users
LEFT JOIN orders
```

Many developers prefer `LEFT JOIN` because keeping the main table on the left makes complex queries easier to read.

---

# 6. FULL JOIN

## Keep Rows from Both Tables

A `FULL JOIN` keeps:

* matching rows
* unmatched rows from the left table
* unmatched rows from the right table

### Syntax

```sql
SELECT columns
FROM table_a
FULL JOIN table_b
    ON table_a.foreign_key = table_b.id;
```

---

## Simple Example

Suppose:

```text
table_a = {1, 2, 3}
table_b = {2, 3, 4}
```

Then:

### INNER JOIN

```text
2, 3
```

Only matching values.

### LEFT JOIN

```text
1, 2, 3
```

Everything from the left table.

### RIGHT JOIN

```text
2, 3, 4
```

Everything from the right table.

### FULL JOIN

```text
1, 2, 3, 4
```

Everything from both tables.

Conceptually:

```text
             A             B

          1  2  3       2  3  4
          │  │  │       │  │  │
          └──┼──┘       └──┼──┘
             │             │
             └──────┬──────┘

INNER →       2  3

LEFT  →     1  2  3

RIGHT →       2  3  4

FULL  →     1  2  3  4
```

---

# 7. FULL JOIN with Users and Orders

We can also use `FULL JOIN` with our store database:

```sql
SELECT
    u.full_name,
    o.id AS order_id
FROM users AS u
FULL JOIN orders AS o
    ON o.user_id = u.id
ORDER BY u.id;
```

With the current schema, this will effectively look like the `LEFT JOIN` result.

Why?

Because:

```sql
orders.user_id
```

is defined as:

```sql
INTEGER NOT NULL REFERENCES users(id)
```

Therefore, every order must reference an existing user.

There cannot normally be an order whose `user_id` points to a nonexistent user.

So there are unmatched users, but no unmatched orders.

---

# 8. Comparing the Four JOIN Types

| JOIN Type    | Keeps unmatched Left Rows | Keeps unmatched Right Rows |
| ------------ | ------------------------: | -------------------------: |
| `INNER JOIN` |                         ❌ |                          ❌ |
| `LEFT JOIN`  |                         ✅ |                          ❌ |
| `RIGHT JOIN` |                         ❌ |                          ✅ |
| `FULL JOIN`  |                         ✅ |                          ✅ |

A simple way to remember:

```text
INNER → Only matches

LEFT  → Everything on the left + matches

RIGHT → Everything on the right + matches

FULL  → Everything from both sides
```

---

# 9. Basic JOIN Examples

## Every Category and Its Products

Even categories without products should appear:

```sql
SELECT
    c.name,
    p.id AS product_id
FROM categories AS c
LEFT JOIN products AS p
    ON p.category_id = c.id
ORDER BY c.id;
```

If a category has no product, `product_id` will be `NULL`.

---

## Completed Orders Without Payments

We can use `LEFT JOIN` to check whether a completed order has a payment record.

```sql
SELECT
    o.id
FROM orders AS o
LEFT JOIN payments AS pay
    ON pay.order_id = o.id
WHERE o.status = 'completed'
  AND pay.id IS NULL;
```

This is a useful **data-quality check**.

If the query returns an order, that completed order has no corresponding payment record.

---

# 10. Realistic Example — Users Who Never Ordered

The marketing team wants to send a welcome discount to users who have never placed an order.

We can solve this using `LEFT JOIN` + `IS NULL`.

```sql
SELECT
    u.full_name,
    u.email
FROM users AS u
LEFT JOIN orders AS o
    ON o.user_id = u.id
WHERE o.id IS NULL
ORDER BY u.full_name;
```

### Expected Result

```text
 full_name        | email
------------------+-----------------------------
 Farzana Rahman   | farzana.rahman@example.com
 Nusrat Jahan     | nusrat.jahan@example.com
 Rashed Karim     | rashed.karim@example.com
```

This is one of the most practical uses of `LEFT JOIN`.

---

# 11. A Reusable Pattern

Whenever the question sounds like:

> "Which records do **not** have a related record?"

Think:

```sql
SELECT parent.*
FROM parent
LEFT JOIN child
    ON child.parent_id = parent.id
WHERE child.id IS NULL;
```

Examples:

```text
Users without orders
Products without order items
Categories without products
Orders without payments
Employees without departments
Students without courses
```

The table names change, but the pattern remains the same.

---

# 12. JOIN Relationship Map

Our CodeBridge store database now supports several types of JOIN queries:

```text
users
  │
  │ user_id
  ▼
orders
  │
  ├──────────────► payments
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

This allows us to ask questions such as:

* Which users have orders?
* Which users have no orders?
* Which categories have no products?
* Which orders have no payments?
* Which products appear in orders?

---

# 13. Key Concepts Learned

* What OUTER JOIN means
* `LEFT JOIN`
* `RIGHT JOIN`
* `FULL JOIN`
* How unmatched rows become `NULL`
* `LEFT JOIN` + `IS NULL`
* Difference between `INNER JOIN` and `LEFT JOIN`
* Why `RIGHT JOIN` can usually be replaced with `LEFT JOIN`
* When `FULL JOIN` is useful
* Using JOINs for data-quality checks
* Finding records without related data

---

# 14. Key Takeaway

> **JOIN type determines which unmatched rows are allowed to remain in the result.**

Remember the four patterns:

```text
INNER JOIN
→ Only matching rows

LEFT JOIN
→ All left rows + matching right rows

RIGHT JOIN
→ All right rows + matching left rows

FULL JOIN
→ All rows from both tables
```

And remember this especially useful pattern:

```sql
LEFT JOIN ... WHERE child.id IS NULL
```

It answers:

> **"Which records don't have a matching record?"**

---
