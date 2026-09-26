# Session 04 — JOIN, WHERE, GROUP BY & Aggregate Functions

**CodeBridge Japan — PostgreSQL Learning**

> Build real business reports by combining multiple tables, filtering rows, grouping data, and calculating totals.

---

## 📋 Session Overview

| Item             | Details                                       |
| ---------------- | --------------------------------------------- |
| **Session**      | 04 / 08                                       |
| **Module**       | Relationships & Real Queries                  |
| **Topic**        | JOIN + WHERE + GROUP BY + Aggregate Functions |
| **Prerequisite** | Session 03                                    |
| **Level**        | Beginner → Intermediate                       |
| **Database**     | PostgreSQL                                    |

---

# 1. What Is This Session About?

This session focuses on combining the SQL concepts learned in Phase 1 with JOINs from Phase 2.

We will use:

* `JOIN` — Combine related tables.
* `WHERE` — Filter individual rows.
* `GROUP BY` — Group rows based on a column.
* `COUNT()` — Count records.
* `SUM()` — Calculate totals.
* `AVG()` — Calculate averages.
* `HAVING` — Filter grouped results.
* `ORDER BY` — Sort the final result.

These are not new SQL keywords. The goal is to learn how to use them together in realistic queries.

---

# 2. Why Do We Need These Queries?

Real business questions often require information from multiple tables.

For example:

> "How much money did each customer spend?"

The required information is spread across:

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
```

* `users` contains customer names.
* `orders` contains order information.
* `order_items` contains quantities and unit prices.

To calculate the total spent by each customer, we need to:

1. Join the related tables.
2. Filter the relevant orders.
3. Group the rows by customer.
4. Calculate the total using `SUM()`.

This is the foundation of real-world business reporting.

---

# 3. SQL Query Execution Order

When JOINs and aggregate functions are used together, PostgreSQL processes the query in a logical order.

```text
FROM + JOIN
      ↓
    WHERE
      ↓
   GROUP BY
      ↓
  Aggregate Functions
      ↓
    HAVING
      ↓
   ORDER BY
      ↓
     LIMIT
```

### Step-by-Step Explanation

| Step                | Purpose                               |
| ------------------- | ------------------------------------- |
| `FROM` + `JOIN`     | Combine related tables                |
| `WHERE`             | Filter individual rows                |
| `GROUP BY`          | Divide rows into groups               |
| Aggregate functions | Calculate totals, counts, or averages |
| `HAVING`            | Filter groups                         |
| `ORDER BY`          | Sort the results                      |
| `LIMIT`             | Restrict the number of returned rows  |

**Important:** This is the logical processing order, not necessarily the exact physical execution plan chosen by PostgreSQL.

---

# 4. JOIN + GROUP BY + Aggregate

## Example 1 — Count Products in Each Category

We want to know how many products belong to each category.

### Query

```sql
SELECT
    c.name,
    COUNT(p.id) AS total_products
FROM categories AS c
INNER JOIN products AS p
    ON p.category_id = c.id
GROUP BY c.name
ORDER BY c.name;
```

### Expected Result

```text
    name     | total_products
-------------+---------------
 Books       |             1
 Gear        |             3
 Study Aids  |             1
```

### Explanation

* `JOIN` connects categories with products.
* `COUNT(p.id)` counts the matching products.
* `GROUP BY c.name` creates one group per category.
* `ORDER BY c.name` sorts the category names.

**Note:** Because this uses `INNER JOIN`, categories without products will not appear.

---

# 5. JOIN + WHERE + GROUP BY + HAVING

## Example 2 — Total Spending by Customer

The finance team wants to know how much each customer spent on completed orders.

### Query

```sql
SELECT
    u.full_name,
    SUM(oi.unit_price_cents * oi.quantity) AS total_spent_cents
FROM users AS u
INNER JOIN orders AS o
    ON o.user_id = u.id
INNER JOIN order_items AS oi
    ON oi.order_id = o.id
WHERE o.status = 'completed'
GROUP BY u.id, u.full_name
ORDER BY total_spent_cents DESC;
```

### Expected Result

```text
    full_name     | total_spent_cents
------------------+------------------
 Mitsuki Sato     |             8999
 Kenji Watanabe   |             6499
 Tanvir Ahmed     |             3997
```

### Explanation

**1. JOIN**

Connects users, orders, and order items.

**2. WHERE**

Keeps only completed orders.

```sql
WHERE o.status = 'completed'
```

**3. GROUP BY**

Groups the rows by customer.

```sql
GROUP BY u.id, u.full_name
```

**4. SUM()**

Calculates the total spending in cents.

```sql
SUM(oi.unit_price_cents * oi.quantity)
```

**5. ORDER BY**

Displays customers from highest to lowest spending.

---

## Why Use Cents?

The `unit_price_cents` column stores prices as integers.

For example:

```text
1999 cents = ¥19.99
```

Using integer cents avoids many floating-point rounding issues during monetary calculations.

To display the total in currency units:

```sql
ROUND(
    SUM(oi.unit_price_cents * oi.quantity) / 100.0,
    2
) AS total_spent
```

---

# 6. JOIN + LEFT JOIN + GROUP BY

## Example 3 — Total Orders per User

We want to count how many orders each user has placed, including users who have never placed an order.

### Query

```sql
SELECT
    u.id,
    u.full_name,
    COUNT(o.id) AS total_orders
FROM users AS u
LEFT JOIN orders AS o
    ON o.user_id = u.id
GROUP BY u.id, u.full_name
ORDER BY total_orders DESC, u.id;
```

### Expected Result

```text
 id | full_name        | total_orders
----+------------------+-------------
  1 | Tanvir Ahmed     |            2
  2 | Mitsuki Sato     |            1
  4 | Kenji Watanabe   |            1
  6 | Aiko Tanaka      |            1
  3 | Farzana Rahman   |            0
  5 | Rashed Karim     |            0
  7 | Nusrat Jahan     |            0
```

### Important Concept: COUNT(o.id) vs COUNT(*)

With a `LEFT JOIN`, users without orders still produce a result row with `NULL` values from the orders table.

Therefore:

```sql
COUNT(o.id)
```

counts only actual matching orders.

But:

```sql
COUNT(*)
```

counts the joined row itself, even when no order exists.

For this report, use:

```sql
COUNT(o.id)
```

to get zero for users without orders.

---

# 7. Aggregate Functions with JOIN

Aggregate functions can be used after joining tables.

| Function  | Purpose               |
| --------- | --------------------- |
| `COUNT()` | Count records         |
| `SUM()`   | Calculate the total   |
| `AVG()`   | Calculate the average |
| `MIN()`   | Find the minimum      |
| `MAX()`   | Find the maximum      |

### Example — Total Quantity Sold per Product

```sql
SELECT
    p.name,
    SUM(oi.quantity) AS total_sold
FROM order_items AS oi
INNER JOIN products AS p
    ON p.id = oi.product_id
GROUP BY p.id, p.name
ORDER BY total_sold DESC;
```

This report shows how many units of each product were included in the orders.

---

# 8. Subquery Introduction — Average Items per Order

Sometimes we need to calculate an aggregate result from another query.

This is called a **subquery**.

The following example first counts the items in each order, then calculates the average count across orders.

```sql
SELECT
    ROUND(AVG(item_counts.total), 1) AS avg_items_per_order
FROM (
    SELECT
        order_id,
        COUNT(*) AS total
    FROM order_items
    GROUP BY order_id
) AS item_counts;
```

### How It Works

The inner query:

```sql
SELECT
    order_id,
    COUNT(*) AS total
FROM order_items
GROUP BY order_id;
```

calculates the number of line items in each order.

The outer query:

```sql
AVG(item_counts.total)
```

calculates the average across those orders.

**Note:** This counts order-item rows, not the total number of physical units. To count units, use `SUM(quantity)` instead.

Subqueries will be covered in more detail in a later session.

---

# 9. Real Business Report — Best-Selling Products

The finance team has two questions:

1. How much did each customer spend?
2. Which products sold the most units?

We already answered the first question.

Now let's build the second report.

### Query

```sql
SELECT
    p.name,
    SUM(oi.quantity) AS total_sold
FROM order_items AS oi
INNER JOIN products AS p
    ON p.id = oi.product_id
GROUP BY p.id, p.name
ORDER BY total_sold DESC, p.name;
```

### Expected Result

```text
              name               | total_sold
---------------------------------+-----------
 PostgreSQL Cheat Sheet Poster   |          3
 Beginner SQL Handbook           |          2
 CodeBridge Hoodie               |          1
 Desk Lamp                       |          1
 Mechanical Keyboard             |          1
```

The PostgreSQL Cheat Sheet Poster has the highest quantity sold in the current sample data.

---

# 10. HAVING — Filter Groups

`WHERE` filters individual rows before grouping.

`HAVING` filters groups after aggregation.

### Example — Customers Who Spent More Than 5,000 Cents

```sql
SELECT
    u.full_name,
    SUM(oi.unit_price_cents * oi.quantity) AS total_spent_cents
FROM users AS u
INNER JOIN orders AS o
    ON o.user_id = u.id
INNER JOIN order_items AS oi
    ON oi.order_id = o.id
WHERE o.status = 'completed'
GROUP BY u.id, u.full_name
HAVING SUM(oi.unit_price_cents * oi.quantity) > 5000
ORDER BY total_spent_cents DESC;
```

### Difference Between WHERE and HAVING

| Clause   | Filters                  |
| -------- | ------------------------ |
| `WHERE`  | Individual rows          |
| `HAVING` | Groups after aggregation |

---

# 11. Key Concepts Learned

* Combining JOINs with aggregate functions
* Using `WHERE` before grouping
* Grouping joined data with `GROUP BY`
* Calculating totals with `SUM()`
* Counting records with `COUNT()`
* Calculating averages with `AVG()`
* Filtering groups with `HAVING`
* Sorting aggregated results with `ORDER BY`
* Using `LEFT JOIN` to include users with zero orders
* Understanding `COUNT(*)` vs `COUNT(column)`
* Introduction to subqueries
* Creating customer spending reports
* Creating best-selling product reports

---

# 12. Key Takeaway

> **JOIN combines related data, WHERE filters rows, GROUP BY creates groups, and aggregate functions calculate results for each group.**

A common business-reporting pattern is:

```sql
SELECT
    grouping_column,
    SUM(value_column) AS total
FROM table_a AS a
INNER JOIN table_b AS b
    ON b.a_id = a.id
WHERE condition
GROUP BY grouping_column
HAVING SUM(value_column) > 0
ORDER BY total DESC;
```

This pattern is useful for customer spending, sales reports, product performance, and many other business questions.

---
