# Phase 3 – Session 04: Views

**CodeBridge Japan – PostgreSQL Learning Journey**

## Session Overview

| Item         | Details                                                              |
| ------------ | -------------------------------------------------------------------- |
| Phase        | Phase 3 – Advanced Queries & Real-World Patterns                     |
| Session      | 04 / 08                                                              |
| Topic        | Views                                                                |
| Concepts     | `CREATE VIEW`, `SELECT`, `CREATE OR REPLACE VIEW`, `DROP VIEW`, CTEs |
| Prerequisite | Session 03 – Window Functions II                                     |
| Level        | Beginner to Intermediate                                             |

---

## 1. Introduction to Views

A **view** is a named SQL query stored in the database. It behaves like a virtual table and allows users to access the results of a query without rewriting the entire SQL statement.

A standard PostgreSQL view stores the query definition, not a separate copy of the underlying data. When queried, PostgreSQL retrieves results from the underlying tables using the view's definition.

### Views vs. CTEs

| Feature                        | CTE                        | View                                              |
| ------------------------------ | -------------------------- | ------------------------------------------------- |
| Definition                     | Named query using `WITH`   | Named query created using `CREATE VIEW`           |
| Lifetime                       | One SQL statement          | Persists in the database until changed or dropped |
| Reusability                    | Within the statement       | Across multiple queries and sessions              |
| Stores a separate copy of data | No                         | No                                                |
| Typical use                    | Organizing complex queries | Reusing common queries and reports                |

---

## 2. Why Views Are Useful

Suppose a finance manager needs the same revenue or spending report every week.

Without a view, the SQL query must be copied and executed repeatedly. This can make reporting time-consuming and increase the risk of mistakes.

A view solves this problem by allowing users to:

* Save a complex query under a meaningful name.
* Reuse the query without rewriting it.
* Simplify access to frequently used reports.
* Make complex SQL easier for other team members to query.
* Keep report logic consistent across multiple queries.

---

## 3. How Views Work

A standard view stores the SQL query definition rather than a snapshot of its results.

When someone executes:

```sql
SELECT *
FROM user_spending;
```

PostgreSQL uses the view's stored query definition to retrieve data from the underlying tables.

### Important Characteristics

* A standard view does not store a separate copy of the query results.
* Changes to the underlying tables can be reflected the next time the view is queried.
* The view remains available across database sessions.
* Access to a view depends on database permissions and the permissions required to access its underlying objects.

**Key takeaway:** A view is a reusable, named query that provides a convenient way to access underlying data.

---

## 4. Creating a View with CREATE VIEW

### Syntax

```sql
CREATE VIEW view_name AS
SELECT ...
FROM table_name;
```

### Example: Create a User Spending View

This view calculates the total amount spent by each customer on completed orders.

```sql
CREATE VIEW user_spending AS
SELECT
    u.full_name,
    SUM(oi.unit_price_cents * oi.quantity) AS total_spent_cents
FROM users AS u
JOIN orders AS o
    ON o.user_id = u.id
JOIN order_items AS oi
    ON oi.order_id = o.id
WHERE o.status = 'completed'
GROUP BY u.full_name;
```

### Using the View

Once created, the view can be queried like a table.

```sql
SELECT *
FROM user_spending
WHERE total_spent_cents > (
    SELECT AVG(total_spent_cents)
    FROM user_spending
)
ORDER BY total_spent_cents DESC;
```

### Expected Output

| full_name      | total_spent_cents |
| -------------- | ----------------: |
| Mitsuki Sato   |              8999 |
| Kenji Watanabe |              6499 |

**Key observation:** The view stores the spending query definition, so the same logic can be reused without writing the joins and aggregation again.

---

## 5. Modifying and Deleting Views

PostgreSQL provides commands to update or remove a view.

### CREATE OR REPLACE VIEW

Use `CREATE OR REPLACE VIEW` to update the query definition of an existing view.

```sql
CREATE OR REPLACE VIEW view_name AS
SELECT ...
FROM table_name;
```

The replacement query must preserve the existing columns' names, order, and data types for existing columns. Additional columns can be appended at the end.

### DROP VIEW

Use `DROP VIEW` to remove a view from the database.

```sql
DROP VIEW view_name;
```

To avoid an error if the view does not exist:

```sql
DROP VIEW IF EXISTS view_name;
```

**Important:** Dropping a view removes the view definition, not the underlying table data.

---

## 6. Basic Example: Product Catalog View

A product catalog view combines product names, category names, and formatted prices.

### Create the View

```sql
CREATE VIEW product_catalog AS
SELECT
    p.name,
    c.name AS category,
    ROUND(p.price_cents / 100.0, 2) AS price_dollars
FROM products AS p
JOIN categories AS c
    ON c.id = p.category_id;
```

### Query the View

```sql
SELECT *
FROM product_catalog
WHERE category = 'Gear'
ORDER BY price_dollars DESC;
```

### Expected Output

| name                | category | price_dollars |
| ------------------- | -------- | ------------: |
| Mechanical Keyboard | Gear     |         89.99 |
| CodeBridge Hoodie   | Gear     |         45.00 |
| Desk Lamp           | Gear     |         24.99 |

**Key observation:** The view simplifies product queries by hiding the underlying join and price-conversion logic.

---

## 7. Real-World Example: Finance Manager's Revenue Growth Report

### Business Scenario

The finance manager wants a revenue growth report every Monday morning.

Instead of repeatedly running the full query from Session 03, we can save the running revenue calculation as a view.

### Create the Revenue Growth View

```sql
CREATE VIEW revenue_growth AS
WITH running_revenue AS (
    SELECT
        paid_at,
        amount_cents,
        SUM(amount_cents) OVER (
            ORDER BY paid_at
        ) AS running_total_cents
    FROM payments
)
SELECT
    paid_at,
    amount_cents,
    running_total_cents,
    ROUND(
        100.0 * running_total_cents /
        (
            SELECT MAX(running_total_cents)
            FROM running_revenue
        ),
        1
    ) AS percent_of_total
FROM running_revenue;
```

### Query the Report

```sql
SELECT *
FROM revenue_growth
ORDER BY paid_at;
```

### Expected Output

| paid_at                | amount_cents | running_total_cents | percent_of_total |
| ---------------------- | -----------: | ------------------: | ---------------: |
| 2024-04-01 10:05:00+09 |         3997 |                3997 |             20.5 |
| 2024-04-03 11:35:00+09 |         8999 |               12996 |             66.7 |
| 2024-04-12 14:05:00+09 |         6499 |               19495 |            100.0 |

### How It Works

1. The `running_revenue` CTE calculates cumulative revenue using `SUM()`.
2. The outer query calculates the percentage of the final running total.
3. `CREATE VIEW` saves the complete query under the name `revenue_growth`.
4. The finance manager can retrieve the report using a simple `SELECT` statement.

**Key observation:** CTEs and window functions can be used inside a view, allowing complex analytical queries to be saved and reused.

---

## 8. Key Concepts Learned

* A view is a named SQL query stored in the database.
* Standard views do not store a separate copy of the underlying data.
* Views can be queried like tables using `SELECT`.
* `CREATE VIEW` creates a reusable query definition.
* `CREATE OR REPLACE VIEW` updates an existing view's definition.
* `DROP VIEW` removes a view from the database.
* Views can simplify complex joins, aggregations, and analytical queries.
* CTEs and window functions can be used inside a view.
* Changes to underlying data can be reflected when a standard view is queried again.

---

## 9. Session Summary

| Command / Concept              | Purpose                              |
| ------------------------------ | ------------------------------------ |
| `CREATE VIEW`                  | Creates a named query                |
| `SELECT * FROM view_name`      | Retrieves results from a view        |
| `CREATE OR REPLACE VIEW`       | Updates a view definition            |
| `DROP VIEW`                    | Removes a view                       |
| CTE inside a view              | Organizes complex query logic        |
| Window functions inside a view | Supports reusable analytical reports |

### Key Takeaway

**A view turns a complex SQL query into a reusable database object, making recurring reports easier to access and maintain.**

---


