# Session 08 — Bringing It All Together

**CodeBridge Japan | PostgreSQL Learning Journey**

> Final milestone: Building a CodeBridge admin dashboard using the tools learned throughout Phases 1–3.

---

## 📋 Session Overview

| Category         | Details                                      |
| ---------------- | -------------------------------------------- |
| **Phase**        | Phase 3 — Advanced Queries and Real Patterns |
| **Session**      | 08 / 08                                      |
| **Topic**        | Bringing It All Together                     |
| **Level**        | Beginner — Intermediate                      |
| **Prerequisite** | Session 07 — Role and Permission Basics      |
| **Database**     | PostgreSQL                                   |

---

## 📖 1. Where the Story Left Off

Rafi's manager calls a short meeting.

> "You've created many one-time reports for us this year — revenue charts, customer lists, and product rankings. They're all useful, but they're scattered across chats and old messages. Before we declare Phase 3 complete, I want something permanent: an admin dashboard."

Rafi realizes that this is not about learning a completely new set of SQL tools.

It is about combining everything he has already learned into a practical database deliverable.

The team decides to build three permanent database views that answer recurring business questions.

---

## 🎯 2. Learning Objectives

By the end of this session, I learned how to:

* Combine SQL concepts from Phases 1–3 into a practical deliverable.
* Use `CASE WHEN` for conditional logic.
* Create permanent database views using `CREATE VIEW`.
* Build a customer overview using `LEFT JOIN`, CTEs, and aggregates.
* Create a complete monthly revenue report using `generate_series()`.
* Rank products within their categories using `RANK()`.
* Grant explicit `SELECT` permissions on dashboard views.
* Organize reusable business reports into a simple admin dashboard.

---

## 🧠 3. What Is an Admin Dashboard?

An **admin dashboard** is a collection of reusable database views that provide answers to recurring business questions.

Instead of running separate queries every time someone needs a report, the team can query saved views directly.

### Dashboard Architecture

| View                            | Main Purpose                                 | Key Concepts                                   |
| ------------------------------- | -------------------------------------------- | ---------------------------------------------- |
| `dashboard_customer_overview`   | Customer orders, spending, and customer type | `LEFT JOIN`, CTE, `GROUP BY`, `CASE WHEN`      |
| `dashboard_monthly_revenue`     | Monthly revenue with no missing months       | `generate_series()`, `LEFT JOIN`, `COALESCE()` |
| `dashboard_product_performance` | Product sales ranking by category            | `JOIN`, `GROUP BY`, `RANK()`                   |

All three views are stored in the database and can be queried by roles that have the necessary permissions.

---

## 🛠️ 4. A New Tool — CASE WHEN

`CASE WHEN` adds conditional logic directly inside a SQL query.

### Syntax

```sql
CASE
    WHEN condition_1 THEN result_1
    WHEN condition_2 THEN result_2
    ELSE default_result
END
```

PostgreSQL evaluates the conditions in order and returns the result for the first matching condition. If no condition matches, it returns the `ELSE` result, or `NULL` if `ELSE` is omitted.

### Example — Classify Products by Price

```sql
SELECT
    name,
    price_cents,
    CASE
        WHEN price_cents > 5000 THEN 'premium'
        WHEN price_cents > 1000 THEN 'standard'
        ELSE 'budget'
    END AS price_tier
FROM products;
```

### Example Output

| name                          | price_cents | price_tier |
| ----------------------------- | ----------: | ---------- |
| Beginner SQL Handbook         |        1999 | standard   |
| PostgreSQL Cheat Sheet Poster |         999 | budget     |
| CodeBridge Hoodie             |        4500 | standard   |
| Mechanical Keyboard           |        8999 | premium    |
| Desk Lamp                     |        2499 | standard   |

---

## 🧪 5. Basic Example — Classify Customers

Use `CASE WHEN` to group customers based on their signup date.

```sql
SELECT
    full_name,
    CASE
        WHEN created_at < '2024-02-01' THEN 'early adopter'
        ELSE 'regular signup'
    END AS signup_group
FROM users
ORDER BY created_at;
```

This query labels customers as either early adopters or regular signups.

---

## 💼 6. Real-World Example — Build Three Dashboard Views

### View 1: Customer Overview

**Business Question:** Who are the repeat customers, how many orders have they placed, and how much have they spent?

This view combines:

* `LEFT JOIN` to keep customers without orders.
* CTEs to organize order counts and spending calculations.
* `GROUP BY` and aggregate functions to summarize customer activity.
* `COALESCE()` to display zero when no completed spending exists.
* `CASE WHEN` to classify customers.

### SQL Solution

```sql
CREATE VIEW dashboard_customer_overview AS
WITH order_counts AS (
    SELECT
        u.id,
        u.full_name,
        u.email,
        COUNT(o.id) AS total_orders
    FROM users AS u
    LEFT JOIN orders AS o
        ON o.user_id = u.id
    GROUP BY
        u.id,
        u.full_name,
        u.email
),
completed_spend AS (
    SELECT
        o.user_id,
        SUM(oi.unit_price_cents * oi.quantity) AS total_spent_cents
    FROM orders AS o
    JOIN order_items AS oi
        ON oi.order_id = o.id
    WHERE o.status = 'completed'
    GROUP BY o.user_id
)
SELECT
    oc.full_name,
    oc.email,
    oc.total_orders,
    COALESCE(cs.total_spent_cents, 0) AS total_spent_cents,
    CASE
        WHEN oc.total_orders = 0 THEN 'no orders yet'
        WHEN oc.total_orders = 1 THEN 'first-time'
        ELSE 'repeat'
    END AS customer_type
FROM order_counts AS oc
LEFT JOIN completed_spend AS cs
    ON cs.user_id = oc.id;
```

### Example Output

| full_name      | total_orders | total_spent_cents | customer_type |
| -------------- | -----------: | ----------------: | ------------- |
| Aiko Tanaka    |            1 |                 0 | first-time    |
| Farzana Rahman |            0 |                 0 | no orders yet |
| Kenji Watanabe |            1 |              6499 | first-time    |
| Mitsuki Sato   |            1 |              8999 | first-time    |
| Nusrat Jahan   |            0 |                 0 | no orders yet |
| Rashed Karim   |            0 |                 0 | no orders yet |
| Tanvir Ahmed   |            2 |              3997 | repeat        |

Every customer is included, even those who have never placed an order.

**Key takeaway:** `LEFT JOIN` preserves customers without matching order records, while `CASE WHEN` classifies customers based on their order count.

---

### View 2: Monthly Revenue Dashboard

**Business Question:** How much revenue did the business generate each month, including months with no payments?

This view combines:

* `generate_series()` to create a complete calendar.
* CTEs to organize the calendar and monthly revenue.
* `DATE_TRUNC()` to group payments by month.
* `LEFT JOIN` to preserve every calendar month.
* `COALESCE()` to display zero for missing revenue.

### SQL Solution

```sql
CREATE VIEW dashboard_monthly_revenue AS
WITH calendar AS (
    SELECT generate_series(
        '2024-01-01'::timestamptz,
        '2024-12-01'::timestamptz,
        INTERVAL '1 month'
    ) AS month_start
),
monthly_revenue AS (
    SELECT
        DATE_TRUNC('month', paid_at) AS month_start,
        SUM(amount_cents) AS total_cents
    FROM payments
    GROUP BY DATE_TRUNC('month', paid_at)
)
SELECT
    c.month_start,
    COALESCE(mr.total_cents, 0) AS total_cents
FROM calendar AS c
LEFT JOIN monthly_revenue AS mr
    ON mr.month_start = c.month_start;
```

### Query the View

```sql
SELECT *
FROM dashboard_monthly_revenue
ORDER BY month_start;
```

The view covers January through December 2024. Months without matching payment records appear with a revenue value of `0`.

**Key takeaway:** The calendar ensures that every month is represented, regardless of whether payments occurred.

---

### View 3: Product Performance Dashboard

**Business Question:** Which products sell the most within each category?

This view combines:

* `JOIN` to connect products, categories, and order items.
* `GROUP BY` to calculate sales totals.
* `SUM()` to calculate units sold and revenue.
* `RANK()` with `PARTITION BY` to rank products within each category.

### SQL Solution

```sql
CREATE VIEW dashboard_product_performance AS
SELECT
    c.name AS category,
    p.name AS product,
    SUM(oi.quantity) AS total_sold,
    SUM(oi.unit_price_cents * oi.quantity) AS total_revenue_cents,
    RANK() OVER (
        PARTITION BY p.category_id
        ORDER BY SUM(oi.quantity) DESC
    ) AS sales_rank
FROM products AS p
JOIN categories AS c
    ON c.id = p.category_id
JOIN order_items AS oi
    ON oi.product_id = p.id
GROUP BY
    c.name,
    p.category_id,
    p.name;
```

### Query the View

```sql
SELECT *
FROM dashboard_product_performance
ORDER BY category, sales_rank, product;
```

### Example Output

| category   | product                       | total_sold | total_revenue_cents | sales_rank |
| ---------- | ----------------------------- | ---------: | ------------------: | ---------: |
| Books      | Beginner SQL Handbook         |          2 |                3998 |          1 |
| Gear       | Mechanical Keyboard           |          1 |                8999 |          1 |
| Gear       | Desk Lamp                     |          1 |                2499 |          1 |
| Gear       | CodeBridge Hoodie             |          1 |                4500 |          1 |
| Study Aids | PostgreSQL Cheat Sheet Poster |          3 |                2997 |          1 |

The three Gear products are tied with one unit sold each, so `RANK()` assigns each product rank 1.

---

## 🔐 7. Grant Access to the Dashboard Views

After creating the three views, Rafi tries to query the customer overview using the application role.

```sql
SELECT *
FROM dashboard_customer_overview;
```

The application role receives a permission error because the dashboard views were created after the earlier table permissions were granted.

### Solution — Grant SELECT on All Three Views

```sql
GRANT SELECT
ON dashboard_customer_overview,
   dashboard_monthly_revenue,
   dashboard_product_performance
TO codebridge_app;
```

Now the application role can query the three views, subject to the applicable PostgreSQL view and underlying permission rules.

### Verify Access

```sql
SELECT *
FROM dashboard_customer_overview;

SELECT *
FROM dashboard_monthly_revenue
ORDER BY month_start;

SELECT *
FROM dashboard_product_performance
ORDER BY category, sales_rank, product;
```

**Key takeaway:** Permissions granted on existing tables do not automatically apply to views created later. Explicitly grant the required permissions on the new views.

---

## 📚 8. Key Concepts Learned

| Concept             | Purpose                                           |
| ------------------- | ------------------------------------------------- |
| `CASE WHEN`         | Adds conditional logic to SQL queries             |
| `CREATE VIEW`       | Stores a reusable query as a database view        |
| `LEFT JOIN`         | Preserves unmatched rows from the left-side table |
| CTE                 | Organizes complex queries into named result sets  |
| `GROUP BY`          | Groups records for aggregate calculations         |
| `COALESCE()`        | Replaces `NULL` with a specified value            |
| `generate_series()` | Generates a sequence of dates or timestamps       |
| `RANK()`            | Assigns rankings while preserving ties            |
| `PARTITION BY`      | Divides rows into groups for window calculations  |
| `GRANT SELECT`      | Gives a role permission to read a view            |

---

## 💡 9. Key Takeaways

* A database view can turn a complex query into a reusable business report.
* CTEs make multi-step queries easier to understand and maintain.
* `LEFT JOIN` and `COALESCE()` help produce complete reports.
* Window functions can rank products within categories.
* `CASE WHEN` allows business rules to be expressed directly in SQL.
* Permissions must be explicitly granted to application roles for newly created views.
* A dashboard built from reusable views reduces the need to recreate the same reports repeatedly.

---

## ✅ 10. Phase 3 Progress Tracker

* [x] Session 01 — Common Table Expressions (CTE)
* [x] Session 02 — Window Functions I
* [x] Session 03 — Window Functions II
* [x] Session 04 — Views
* [x] Session 05 — Set Operations
* [x] Session 06 — Date and Time Details
* [x] Session 07 — Role and Permission Basics
* [x] Session 08 — Bringing It All Together

---

## 🏆 11. Phase 3 Completion

**Phase 3 — Advanced Queries and Real Patterns: 8 of 8 sessions completed.**

This phase brought together advanced SQL concepts and applied them to a practical database dashboard.

The final deliverable includes three reusable views:

1. `dashboard_customer_overview`
2. `dashboard_monthly_revenue`
3. `dashboard_product_performance`

These views provide a foundation for customer analysis, revenue reporting, and product performance monitoring.

---
