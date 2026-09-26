# Phase 3 – Session 05: Set Operations (UNION, UNION ALL, INTERSECT & EXCEPT)

**CodeBridge Japan – PostgreSQL Learning Journey**

## Session Overview

| Item         | Details                                                       |
| ------------ | ------------------------------------------------------------- |
| Phase        | Phase 3 – Advanced Queries & Real-World Patterns              |
| Session      | 05 / 08                                                       |
| Topic        | Set Operations                                                |
| Concepts     | `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT`, CTEs, `ORDER BY` |
| Prerequisite | Session 04 – Views                                            |
| Level        | Beginner to Intermediate                                      |

---

## 1. Introduction to Set Operations

**Set operations** combine or compare the results of two or more `SELECT` queries.

Unlike `JOIN`, which combines columns from related tables, set operations combine query results vertically by stacking rows or identifying common and differing rows.

### Four Important Set Operations

| Operation   | Description                                                        | Result                             |
| ----------- | ------------------------------------------------------------------ | ---------------------------------- |
| `UNION`     | Combines two result sets and removes duplicates.                   | Unique rows from both queries      |
| `UNION ALL` | Combines two result sets and keeps duplicates.                     | All rows from both queries         |
| `INTERSECT` | Returns rows present in both result sets.                          | Common rows                        |
| `EXCEPT`    | Returns rows from the first query that are absent from the second. | Difference between the result sets |

### Requirements for Set Operations

The queries must satisfy these conditions:

* Both queries must return the same number of columns.
* Corresponding columns must have compatible data types.
* Columns are matched by their position, not by their names.
* The output column names are generally taken from the first query.

---

## 2. Why Set Operations Are Useful

Sometimes two different business conditions are easier to express as separate queries.

For example, a marketing team may want to combine:

* Customers who joined in January or February.
* Customers whose spending is above average.

Instead of combining different conditions into one complicated query, we can write two simple `SELECT` statements and combine their results with `UNION`.

Set operations are useful for:

* Combining customer lists from different conditions.
* Finding customers who satisfy multiple criteria.
* Identifying records that exist in one result but not another.
* Comparing datasets without using joins.
* Creating clean, reusable analytical queries.

---

## 3. Understanding How Set Operations Work

Suppose we have two lists:

**List A:** Tanvir, Mitsuki, Farzana, Kenji, Rashed

**List B:** Mitsuki, Kenji

### Comparing the Results

| Operation   | Result                                          | Number of Rows |
| ----------- | ----------------------------------------------- | -------------: |
| `UNION`     | Tanvir, Mitsuki, Farzana, Kenji, Rashed         |              5 |
| `UNION ALL` | List A followed by List B, including duplicates |              7 |
| `INTERSECT` | Mitsuki, Kenji                                  |              2 |
| `EXCEPT`    | Tanvir, Farzana, Rashed                         |              3 |

**Key observation:** `UNION` removes duplicate rows, while `UNION ALL` preserves them. `INTERSECT` finds common rows, and `EXCEPT` finds rows unique to the first result.

---

## 4. UNION and UNION ALL

### UNION

`UNION` combines the results of two queries and removes duplicate rows.

#### Syntax

```sql
SELECT column_name
FROM table_a

UNION

SELECT column_name
FROM table_b;
```

### Example: Combine Customers from Two Groups

Find users who joined in January or February and customers who spent more than the average.

```sql
SELECT full_name
FROM users
WHERE created_at >= '2024-01-01'
  AND created_at < '2024-03-01'

UNION

SELECT full_name
FROM user_spending
WHERE total_spent_cents > (
    SELECT AVG(total_spent_cents)
    FROM user_spending
);
```

#### Expected Output

| full_name      |
| -------------- |
| Tanvir Ahmed   |
| Mitsuki Sato   |
| Farzana Rahman |
| Kenji Watanabe |
| Rashed Karim   |

The result contains five unique names. Mitsuki and Kenji appear in both lists but are returned only once.

### UNION ALL

`UNION ALL` combines the results of two queries without removing duplicates.

#### Syntax

```sql
SELECT column_name
FROM table_a

UNION ALL

SELECT column_name
FROM table_b;
```

### Example

```sql
SELECT full_name
FROM users
WHERE created_at >= '2024-01-01'
  AND created_at < '2024-03-01'

UNION ALL

SELECT full_name
FROM user_spending
WHERE total_spent_cents > (
    SELECT AVG(total_spent_cents)
    FROM user_spending
);
```

#### Expected Output

| full_name      |
| -------------- |
| Tanvir Ahmed   |
| Mitsuki Sato   |
| Farzana Rahman |
| Kenji Watanabe |
| Rashed Karim   |
| Mitsuki Sato   |
| Kenji Watanabe |

**Key difference:** `UNION ALL` returns seven rows because Mitsuki and Kenji appear twice.

---

## 5. INTERSECT

`INTERSECT` returns only the rows that appear in both query results.

### Syntax

```sql
SELECT column_name
FROM table_a

INTERSECT

SELECT column_name
FROM table_b;
```

### Example: Find Repeat Customers Who Spend Above Average

```sql
WITH numbered_orders AS (
    SELECT
        u.full_name,
        ROW_NUMBER() OVER (
            PARTITION BY o.user_id
            ORDER BY o.created_at
        ) AS order_number
    FROM orders AS o
    JOIN users AS u
        ON u.id = o.user_id
)
SELECT full_name
FROM numbered_orders
WHERE order_number > 1

INTERSECT

SELECT full_name
FROM user_spending
WHERE total_spent_cents > (
    SELECT AVG(total_spent_cents)
    FROM user_spending
);
```

#### Expected Output

```text
(0 rows)
```

**Key observation:** The result is empty because the only repeat customer, Tanvir Ahmed, spent less than the average in the provided dataset.

An empty result is valid: it means no rows were found in both groups.

---

## 6. EXCEPT

`EXCEPT` returns rows from the first query that do not appear in the second query.

### Syntax

```sql
SELECT column_name
FROM table_a

EXCEPT

SELECT column_name
FROM table_b;
```

### Example: Find Users Who Have Never Placed an Order

```sql
SELECT full_name
FROM users

EXCEPT

SELECT u.full_name
FROM users AS u
JOIN orders AS o
    ON o.user_id = u.id;
```

#### Expected Output

| full_name      |
| -------------- |
| Farzana Rahman |
| Nusrat Jahan   |
| Rashed Karim   |

**Key observation:** The query returns customers who have no matching order records.

This answers the same business question as a `LEFT JOIN` combined with `IS NULL`, which was covered in Phase 2.

---

## 7. Basic Examples

### Example 1: Combine Category and Product Names

```sql
SELECT name
FROM categories

UNION

SELECT name
FROM products;
```

This returns a unique list of names from both tables.

### Example 2: Find Gear Products That Have Not Been Purchased

```sql
SELECT p.name
FROM products AS p
JOIN categories AS c
    ON c.id = p.category_id
WHERE c.name = 'Gear'

EXCEPT

SELECT p.name
FROM products AS p
JOIN order_items AS oi
    ON oi.product_id = p.id;
```

This returns products in the Gear category that do not appear in any order item.

---

## 8. Real-World Example: Combined Marketing Contact List

### Business Scenario

The marketing team needs a combined contact list for an email campaign.

The target audience includes:

1. Customers who joined in January or February.
2. Customers whose spending is above average.

We can combine both groups with `UNION` and include each customer's name and email address.

### SQL Query

```sql
SELECT
    full_name,
    email
FROM users
WHERE created_at >= '2024-01-01'
  AND created_at < '2024-03-01'

UNION

SELECT
    u.full_name,
    u.email
FROM users AS u
JOIN user_spending AS us
    ON us.full_name = u.full_name
WHERE us.total_spent_cents > (
    SELECT AVG(total_spent_cents)
    FROM user_spending
)

ORDER BY full_name;
```

### Expected Output

| full_name      | email                                                           |
| -------------- | --------------------------------------------------------------- |
| Farzana Rahman | [farzana.rahman@example.com](mailto:farzana.rahman@example.com) |
| Kenji Watanabe | [kenji.watanabe@example.com](mailto:kenji.watanabe@example.com) |
| Mitsuki Sato   | [mitsuki.sato@example.com](mailto:mitsuki.sato@example.com)     |
| Rashed Karim   | [rashed.karim@example.com](mailto:rashed.karim@example.com)     |
| Tanvir Ahmed   | [tanvir.ahmed@example.com](mailto:tanvir.ahmed@example.com)     |

### How It Works

1. The first query retrieves customers who joined during January and February.
2. The second query retrieves customers whose spending is above average.
3. `UNION` combines both lists and removes duplicate rows.
4. `ORDER BY full_name` sorts the complete combined result alphabetically.

**Important:** The final `ORDER BY` applies to the combined result, not to each individual `SELECT`.

---

## 9. Key Concepts Learned

* Set operations combine or compare the results of separate `SELECT` queries.
* `UNION` combines results and removes duplicate rows.
* `UNION ALL` combines results while preserving duplicates.
* `INTERSECT` returns rows common to both queries.
* `EXCEPT` returns rows from the first query that are absent from the second.
* Both queries must return the same number of columns with compatible data types.
* Set operations match columns by position.
* `ORDER BY` can be used to sort the final combined result.
* Set operations can solve list-comparison problems without using `JOIN`.

---

## 10. Session Summary

| Operation   | Purpose                                    |
| ----------- | ------------------------------------------ |
| `UNION`     | Combine results and remove duplicates      |
| `UNION ALL` | Combine results and preserve duplicates    |
| `INTERSECT` | Find common rows                           |
| `EXCEPT`    | Find rows present only in the first result |
| `ORDER BY`  | Sort the final combined result             |

### Key Takeaway

**Set operations let you combine, compare, and subtract query results without using JOINs. Choose the operation based on whether you need unique rows, duplicates, common records, or differences.**

---
