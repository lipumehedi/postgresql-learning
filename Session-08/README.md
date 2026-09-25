# Session 08 — SQL Functions

> **Module:** Answering Questions
> **Sheet:** 08 / 08
> **Level:** Absolute Beginner
> **Prerequisite:** Session 07 — Aggregation
> **Database:** PostgreSQL

---

## 🎯 Learning Objectives

By the end of this session, I will be able to:

* Transform text using string functions.
* Perform numeric calculations and formatting.
* Extract information from dates and timestamps.
* Handle missing values using `COALESCE` and `NULLIF`.
* Apply conditional logic using `CASE`.
* Format database values for application display.
* Combine SQL functions with filtering and sorting.

---

## 01 — What Are SQL Functions?

### Transforming Values, Not Just Reading Them

A **SQL function** takes one or more values as input and returns a result.

For example, `now()` returns the current timestamp.

PostgreSQL provides several categories of functions:

| Category      | Example                         | Purpose                           |
| ------------- | ------------------------------- | --------------------------------- |
| String        | `UPPER(name)`                   | Transform text                    |
| Numeric       | `ROUND(price, 2)`               | Transform numbers                 |
| Date/Time     | `EXTRACT(year FROM created_at)` | Extract date or time components   |
| NULL Handling | `COALESCE(phone, 'N/A')`        | Replace missing values            |
| Conditional   | `CASE WHEN ... THEN ... END`    | Return values based on conditions |

---

## 02 — Why Do We Need SQL Functions?

Raw database values are not always suitable for displaying directly to users.

Examples:

* A price stored as `1999` cents should appear as `$19.99`.
* A name stored as `aiko tanaka` might need to appear as `AIKO TANAKA`.
* A timestamp can be transformed to display only its year.
* A missing phone number can be displayed as `N/A`.
* A product with zero stock can be labeled `Out of stock`.

SQL functions perform these transformations directly inside queries, making data easier to display and analyze.

---

## 03 — How Do SQL Functions Work?

### One Value In, One Value Out

A function can be applied to each row returned by a query.

Example:

```sql
SELECT
    name,
    ROUND(price_cents / 100.0, 2) AS price_dollars
FROM products;
```

### Example Output

| Product Name                  | Stored Price (Cents) | Display Price |
| ----------------------------- | -------------------: | ------------: |
| Beginner SQL Handbook         |                 1999 |         19.99 |
| PostgreSQL Cheat Sheet Poster |                  999 |          9.99 |
| Mechanical Keyboard           |                 8999 |         89.99 |

The stored `price_cents` value remains unchanged. The function transforms only the value returned by the query.

---

## 04 — Setup: Create the Products Table

This session introduces a new `products` table for practicing numeric, string, and date/time functions.

### Create the Table

```sql
CREATE TABLE products (
    id            SERIAL        PRIMARY KEY,
    name          VARCHAR(200)  NOT NULL,
    price_cents   INTEGER       NOT NULL,
    stock_qty     INTEGER       NOT NULL DEFAULT 0,
    created_at    TIMESTAMPTZ   NOT NULL DEFAULT now()
);
```

### Insert Sample Data

```sql
INSERT INTO products (name, price_cents, stock_qty, created_at) VALUES
    ('Beginner SQL Handbook',         1999, 120, '2024-01-10 10:00:00+09'),
    ('PostgreSQL Cheat Sheet Poster',   999, 300, '2024-01-20 10:00:00+09'),
    ('CodeBridge Hoodie',              4500,   0, '2024-02-05 10:00:00+09'),
    ('Mechanical Keyboard',            8999,  15, '2024-02-25 10:00:00+09'),
    ('Desk Lamp',                      2499,  60, '2024-03-10 10:00:00+09');
```

---

## 05 — String Functions

String functions help transform and manipulate text values.

| Function                             | Purpose                                |   |                     |
| ------------------------------------ | -------------------------------------- | - | ------------------- |
| `UPPER(text)`                        | Convert text to uppercase              |   |                     |
| `LOWER(text)`                        | Convert text to lowercase              |   |                     |
| `LENGTH(text)`                       | Return the number of characters        |   |                     |
| `TRIM(text)`                         | Remove leading and trailing whitespace |   |                     |
| `                                    |                                        | ` | Concatenate strings |
| `SUBSTRING(text FROM start FOR len)` | Extract part of a string               |   |                     |

### Example: Create a Display Label

```sql
SELECT
    full_name || ' <' || email || '>' AS display
FROM users
WHERE id = 1;
```

### Expected Result

```text
Tanvir Ahmed <tanvir.ahmed@example.com>
```

### More Examples

```sql
-- Convert a name to uppercase
SELECT UPPER(full_name)
FROM users
WHERE id = 6;

-- Convert a name to lowercase
SELECT LOWER(full_name)
FROM users
WHERE id = 6;

-- Count the characters in a product name
SELECT LENGTH(name)
FROM products
WHERE id = 1;

-- Remove extra spaces
SELECT TRIM('   PostgreSQL   ');

-- Extract part of a string
SELECT SUBSTRING(name FROM 1 FOR 8)
FROM products
WHERE id = 1;
```

---

## 06 — Numeric Functions

Numeric functions perform calculations and format numerical values.

| Function                  | Purpose                                       |
| ------------------------- | --------------------------------------------- |
| `ROUND(number, decimals)` | Round to a specified number of decimal places |
| `CEIL(number)`            | Round up to the nearest integer               |
| `FLOOR(number)`           | Round down to the nearest integer             |
| `ABS(number)`             | Return the absolute value                     |

### Convert Cents into Dollars

```sql
SELECT
    name,
    ROUND(price_cents / 100.0, 2) AS price_dollars
FROM products
WHERE id = 4;
```

### Expected Result

| Product             | Price |
| ------------------- | ----: |
| Mechanical Keyboard | 89.99 |

### More Examples

```sql
-- Round a number
SELECT ROUND(12.567, 2);

-- Round up
SELECT CEIL(price_cents / 100.0)
FROM products
WHERE id = 4;

-- Round down
SELECT FLOOR(price_cents / 100.0)
FROM products
WHERE id = 4;

-- Return an absolute value
SELECT ABS(-25);
```

---

## 07 — Date and Time Functions

Date and time functions help retrieve and transform information about dates and timestamps.

| Function       | Purpose                                    |
| -------------- | ------------------------------------------ |
| `now()`        | Return the current timestamp               |
| `EXTRACT()`    | Extract a date or time component           |
| `date_trunc()` | Round a timestamp down to a specified unit |
| `AGE()`        | Calculate an interval between timestamps   |

### Extract the Signup Year

```sql
SELECT
    EXTRACT(year FROM created_at) AS signup_year
FROM users
WHERE id = 1;
```

### Extract the Signup Month

```sql
SELECT
    EXTRACT(month FROM created_at) AS signup_month
FROM users
WHERE id = 3;
```

**Expected result:** `2` (February).

### Group Timestamps by Month

```sql
SELECT
    date_trunc('month', created_at) AS signup_month
FROM users;
```

### Calculate Membership Duration

```sql
SELECT
    full_name,
    AGE(created_at) AS member_for
FROM users
WHERE id = 1;
```

The result depends on the current date because `AGE(created_at)` calculates the interval between the current timestamp and the signup timestamp.

---

## 08 — COALESCE, NULLIF, and CASE

These tools help handle missing values and conditional logic.

### COALESCE()

Returns the first non-NULL value from a list of expressions.

```sql
COALESCE(value1, value2, ...)
```

Example:

```sql
SELECT COALESCE(NULL, 'N/A');
```

**Expected result:** `N/A`

Suppose a future version of the `products` table adds an optional `discount_note` column.

```sql
-- Illustration only: discount_note does not exist in our current table.
SELECT
    name,
    COALESCE(discount_note, 'No current discount') AS discount_note
FROM products;
```

### NULLIF()

Returns `NULL` when two expressions are equal. Otherwise, it returns the first expression.

```sql
NULLIF(value1, value2)
```

Example:

```sql
SELECT NULLIF(10, 10);
```

**Expected result:** `NULL`

A practical use is avoiding division by zero:

```sql
SELECT 100.0 / NULLIF(0, 0);
```

The result is `NULL` rather than a division-by-zero error.

### CASE

`CASE` returns a value based on one or more conditions.

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE fallback_result
END
```

### Example: Product Availability

```sql
SELECT
    name,
    CASE
        WHEN stock_qty = 0 THEN 'Out of stock'
        ELSE 'In stock'
    END AS availability
FROM products
WHERE id = 3;
```

### Expected Result

| Product           | Availability |
| ----------------- | ------------ |
| CodeBridge Hoodie | Out of stock |

---

## 09 — Basic Examples

### Convert a Name to Uppercase

```sql
SELECT UPPER(full_name)
FROM users
WHERE id = 6;

-- AIKO TANAKA
```

### Count Characters in a Product Name

```sql
SELECT LENGTH(name)
FROM products
WHERE id = 1;

-- 21
```

### Use CEIL and FLOOR

```sql
SELECT
    CEIL(price_cents / 100.0),
    FLOOR(price_cents / 100.0)
FROM products
WHERE id = 4;

-- 90, 89
```

### Extract a Month

```sql
SELECT EXTRACT(month FROM created_at) AS signup_month
FROM users
WHERE id = 3;

-- 2
```

---

## 10 — Realistic Example: Product Catalog Listing

### Scenario

An admin dashboard needs to display every product with:

* A readable price in dollars.
* A stock availability label.
* Products sorted from highest to lowest price.

### SQL Query

```sql
SELECT
    name,
    ROUND(price_cents / 100.0, 2) AS price_dollars,
    CASE
        WHEN stock_qty = 0 THEN 'Out of stock'
        WHEN stock_qty < 20 THEN 'Low stock'
        ELSE 'In stock'
    END AS availability
FROM products
ORDER BY price_dollars DESC;
```

### Expected Result

| Product                       | Price | Availability |
| ----------------------------- | ----: | ------------ |
| Mechanical Keyboard           | 89.99 | Low stock    |
| CodeBridge Hoodie             | 45.00 | Out of stock |
| Desk Lamp                     | 24.99 | In stock     |
| Beginner SQL Handbook         | 19.99 | In stock     |
| PostgreSQL Cheat Sheet Poster |  9.99 | In stock     |

This query combines three concepts:

1. `ROUND()` converts cents into a readable price.
2. `CASE` converts stock quantities into descriptive labels.
3. `ORDER BY` sorts products by price, highest first.

---

## 12 — Practice Exercises

Try writing these SQL queries yourself.

* [ ] Convert all user names to uppercase.
* [ ] Convert all user emails to lowercase.
* [ ] Find the length of each product name.
* [ ] Display all product prices in dollars.
* [ ] Find the rounded-up and rounded-down prices of every product.
* [ ] Extract the signup year and month for every user.
* [ ] Calculate the membership duration of each user.
* [ ] Use `COALESCE()` to provide a fallback for a NULL value.
* [ ] Use `NULLIF()` to avoid division by zero.
* [ ] Use `CASE` to label products as in stock, low stock, or out of stock.
* [ ] Build a product catalog query that combines price formatting, stock status, and sorting.

---

## 🧠 Key Takeaways

* SQL functions transform values and return results.
* String functions manipulate text.
* Numeric functions perform calculations and formatting.
* Date/time functions extract and transform timestamps.
* `COALESCE()` provides a fallback for NULL values.
* `NULLIF()` returns NULL when two values are equal.
* `CASE` implements conditional logic.
* Functions can be combined with `WHERE`, `ORDER BY`, and other SQL clauses.

---

## 🛠️ Technologies & Concepts

* PostgreSQL
* SQL
* String Functions
* Numeric Functions
* Date/Time Functions
* `COALESCE()`
* `NULLIF()`
* `CASE`
* Data Formatting
* Conditional Logic
---
