# Session 01 — CTE (Common Table Expression)

**CodeBridge Japan — PostgreSQL Learning**

## Session Overview

| Topic         | Details                                            |
| ------------- | -------------------------------------------------- |
| Module        | Advanced Queries and Real Patterns                 |
| Phase         | Phase 3                                            |
| Session       | 01 / 08                                            |
| Prerequisites | Phase 2 Completed                                  |
| Level         | Beginner — Intermediate                            |
| Main Topics   | CTE, WITH, Query Readability, Reusable Query Logic |

---

## 1. What Is a CTE?

**CTE (Common Table Expression)** allows you to assign a name to a query result and use that result in a larger SQL statement.

A CTE is defined using the `WITH` clause before the main query.

### Basic Syntax

```sql
WITH named_result AS (
    SELECT ...
)
SELECT *
FROM named_result;
```

A CTE behaves like a temporary named result that can be referenced within the statement where it is defined.

You can use it to:

* Make complex queries easier to read.
* Break a large query into smaller logical steps.
* Reuse a named result within the same query.
* Apply filtering, joins, and aggregate functions to intermediate results.

**Key idea:** A CTE improves query organization and readability. It does not automatically make a query faster than an equivalent subquery.

---

## 2. Why Are CTEs Necessary?

### The Problem with Repeated Subqueries

Imagine the Finance team asks:

> Show customers who spent more than the average customer, sorted from highest spending to lowest.

To solve this, we need to:

1. Calculate the total spending for each customer.
2. Calculate the average spending across customers.
3. Select customers whose spending is greater than that average.
4. Sort the results in descending order.

Without a CTE, the customer spending calculation may need to be repeated in multiple subqueries.

This creates several problems:

* The SQL becomes longer and harder to read.
* The same calculation is repeated.
* Updating the logic in multiple places increases the risk of mistakes.

A CTE allows us to define the customer spending calculation once and reference it by name.

---

## 3. How Does a CTE Work?

A CTE is defined in the `WITH` clause and then referenced by the main query.

### Example Structure

```sql
WITH user_spending AS (
    SELECT
        full_name,
        SUM(oi.unit_price_cents * oi.quantity) AS total_spent_cents
    FROM users AS u
    JOIN orders AS o
        ON o.user_id = u.id
    JOIN order_items AS oi
        ON oi.order_id = o.id
    WHERE o.status = 'completed'
    GROUP BY u.full_name
)
SELECT *
FROM user_spending;
```

### CTE Workflow

```text
WITH user_spending AS (...)
          |
          v
Create a named query result
          |
          v
Main SELECT references user_spending
          |
          v
Filter, sort, or aggregate the result
          |
          v
Return the final output
```

A CTE name is available only within the SQL statement where it is defined.

**Important:** PostgreSQL may inline a CTE or materialize it depending on the query and its execution plan. A CTE should not be assumed to execute exactly once in every situation.

---

## 4. CTE Syntax

### 4.1 Single CTE

```sql
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT *
FROM cte_name
WHERE another_condition;
```

### 4.2 Example — Products Above the Average Price

This example uses a CTE to organize the product price calculation.

```sql
WITH product_prices AS (
    SELECT
        name,
        price_cents
    FROM products
)
SELECT
    name,
    price_cents
FROM product_prices
WHERE price_cents > (
    SELECT AVG(price_cents)
    FROM product_prices
);
```

### Example Output

| name                | price_cents |
| ------------------- | ----------: |
| CodeBridge Hoodie   |        4500 |
| Mechanical Keyboard |        8999 |

The exact output depends on the data in the database.

The important point is that the product price result has a meaningful name: `product_prices`.

---

## 5. Multiple CTEs

You can define multiple CTEs within a single `WITH` clause.

Separate each CTE with a comma.

### Syntax

```sql
WITH first_cte AS (
    SELECT ...
),
second_cte AS (
    SELECT ...
    FROM first_cte
    WHERE ...
),
third_cte AS (
    SELECT ...
    FROM second_cte
)
SELECT *
FROM third_cte;
```

A later CTE can reference an earlier CTE in the same `WITH` clause.

### Why Use Multiple CTEs?

Multiple CTEs can divide a complex query into clear steps:

1. Extract the required data.
2. Calculate intermediate results.
3. Apply additional filtering or aggregation.
4. Return the final output.

This makes complex SQL easier to understand and maintain.

---

## 6. Basic Example — Products in the Gear Category

Let's retrieve products belonging to the `Gear` category and display their prices in dollars.

```sql
WITH gear_products AS (
    SELECT
        p.name,
        p.price_cents
    FROM products AS p
    JOIN categories AS c
        ON c.id = p.category_id
    WHERE c.name = 'Gear'
)
SELECT
    name,
    ROUND(price_cents / 100.0, 2) AS price_dollars
FROM gear_products
ORDER BY price_dollars DESC;
```

### Example Output

| name                | price_dollars |
| ------------------- | ------------: |
| Mechanical Keyboard |         89.99 |
| CodeBridge Hoodie   |         45.00 |
| Desk Lamp           |         24.99 |

### Explanation

* `gear_products` stores the result of the category filter and join.
* The main query converts prices from cents to dollars.
* `ORDER BY` sorts the products from highest price to lowest.

---

## 7. Real-World Example — Customer Spending Above Average

### Business Requirement

The Finance team wants a report showing customers who spent more than the average customer, sorted by highest spending.

Only completed orders should be included.

### 7.1 The Repeated Subquery Approach

Without a CTE, the customer spending calculation may need to be repeated.

```sql
SELECT
    full_name,
    total_spent_cents
FROM (
    SELECT
        u.full_name,
        SUM(oi.unit_price_cents * oi.quantity) AS total_spent_cents
    FROM users AS u
    JOIN orders AS o
        ON o.user_id = u.id
    JOIN order_items AS oi
        ON oi.order_id = o.id
    WHERE o.status = 'completed'
    GROUP BY u.id, u.full_name
) AS spend
WHERE total_spent_cents > (
    SELECT AVG(total_spent_cents)
    FROM (
        SELECT
            SUM(oi.unit_price_cents * oi.quantity) AS total_spent_cents
        FROM orders AS o
        JOIN order_items AS oi
            ON oi.order_id = o.id
        WHERE o.status = 'completed'
        GROUP BY o.user_id
    ) AS spend_again
)
ORDER BY total_spent_cents DESC;
```

### Limitations

The query repeats the customer spending calculation.

If the Finance team changes the order status filter, the logic may need to be updated in multiple places.

This makes the query harder to maintain.

---

### 7.2 The CTE Approach

Now let's define the customer spending calculation once and reuse it.

```sql
WITH user_spending AS (
    SELECT
        u.id AS user_id,
        u.full_name,
        SUM(oi.unit_price_cents * oi.quantity) AS total_spent_cents
    FROM users AS u
    JOIN orders AS o
        ON o.user_id = u.id
    JOIN order_items AS oi
        ON oi.order_id = o.id
    WHERE o.status = 'completed'
    GROUP BY u.id, u.full_name
)
SELECT
    full_name,
    total_spent_cents
FROM user_spending
WHERE total_spent_cents > (
    SELECT AVG(total_spent_cents)
    FROM user_spending
)
ORDER BY total_spent_cents DESC;
```

### Example Output

| full_name      | total_spent_cents |
| -------------- | ----------------: |
| Mitsuki Sato   |              8999 |
| Kenji Watanabe |              6499 |

These results assume the sample customer spending values provided in the lesson.

### Understanding the Query

**Step 1 — Define the CTE**

`user_spending` calculates the total spending of each customer with completed orders.

**Step 2 — Calculate the Average**

The subquery calculates the average of `total_spent_cents` from `user_spending`.

**Step 3 — Filter Customers**

The `WHERE` clause keeps only customers whose total spending exceeds the average.

**Step 4 — Sort the Results**

`ORDER BY total_spent_cents DESC` sorts the customers from highest spending to lowest.

### Why Is This Better?

The customer spending calculation is written once.

If the Finance team changes the order status filter, it only needs to be updated in the CTE definition.

This reduces duplication and improves readability.

---

## 8. Subquery vs. CTE

| Feature     | Subquery                                | CTE                                           |
| ----------- | --------------------------------------- | --------------------------------------------- |
| Syntax      | Nested inside another query             | Defined using `WITH`                          |
| Readability | Can become difficult in complex queries | Often easier to read in multiple steps        |
| Reuse       | May require repeating the logic         | Can reference the named result multiple times |
| Performance | Depends on the query and execution plan | Depends on the query and execution plan       |
| Best use    | Simple, focused nested queries          | Complex queries with multiple logical steps   |

**Key takeaway:** Choose the approach that makes the query easiest to understand and maintain. Neither approach is automatically faster in every case.

---

## 9. Practice Exercises

### Exercise 1 — Create a Basic CTE

1. Create a CTE named `all_products`.
2. Select product names and prices from `products`.
3. Use the main query to display the result.

### Exercise 2 — Filter Products by Category

1. Create a CTE named `gear_products`.
2. Join `products` with `categories`.
3. Filter products belonging to the `Gear` category.
4. Sort products by price in descending order.

### Exercise 3 — Calculate Customer Spending

1. Create a CTE named `user_spending`.
2. Calculate total spending for each customer.
3. Include only completed orders.
4. Display each customer's total spending.

### Exercise 4 — Find Customers Above Average

1. Reuse the `user_spending` CTE.
2. Calculate the average customer spending.
3. Filter customers whose spending exceeds the average.
4. Sort the result from highest to lowest spending.

### Exercise 5 — Use Multiple CTEs

1. Create a CTE for completed orders.
2. Create a second CTE to calculate customer spending.
3. Use the final query to display customers spending more than the average.

---

## 10. Key Concepts Learned

* What a Common Table Expression (CTE) is.
* How to define a CTE using the `WITH` clause.
* How to reference a named query result in a main query.
* How to create multiple CTEs in one statement.
* How to use a CTE with joins, filtering, and aggregate functions.
* How CTEs reduce repeated query logic.
* How to use CTEs for customer spending reports.
* The differences between subqueries and CTEs.
* Why CTEs improve readability and maintainability.

---

## 11. Key Takeaway

**A CTE gives a query result a meaningful name, making complex SQL easier to read, organize, and reuse.**

Use `WITH` to define intermediate results, then reference those results in the main query.

CTEs are especially useful for financial reports, customer analytics, and queries involving multiple calculation steps.

---
