# Phase 3 – Session 03: Window Functions II (Running Total & Moving Statistics)

**CodeBridge Japan – PostgreSQL Learning Journey**

## Session Overview

| Item         | Details                                                                         |
| ------------ | ------------------------------------------------------------------------------- |
| Phase        | Phase 3 – Advanced Queries & Real-World Patterns                                |
| Session      | 03 / 08                                                                         |
| Topic        | Running Totals and Moving Statistics                                            |
| Concepts     | `SUM()`, `AVG()`, `COUNT()`, `OVER`, `ORDER BY`, `PARTITION BY`, `ROWS BETWEEN` |
| Prerequisite | Session 02 – Window Functions I                                                 |
| Level        | Beginner to Intermediate                                                        |

---

## 1. Introduction to Running Totals and Moving Statistics

A **running total**, also known as a cumulative total, calculates the sum of the current row and all preceding rows in a specified order.

A **moving average** calculates the average over a limited window of rows, such as the current row and the one immediately before it.

Both can be created using aggregate functions such as `SUM()` and `AVG()` as window functions.

### Key Concepts

| Concept        | Description                                                             |
| -------------- | ----------------------------------------------------------------------- |
| Running Total  | Accumulates values from the beginning of a window to the current row.   |
| Moving Average | Calculates the average over a moving frame of rows.                     |
| Window Frame   | Defines which rows are included in the calculation for the current row. |
| `ROWS BETWEEN` | Specifies the frame boundaries using physical row positions.            |

---

## 2. Why Running Totals Are Useful

A final revenue total tells us how much money a business earned, but it does not show how revenue accumulated over time.

A running total helps answer questions such as:

* How much revenue had been collected by a particular payment date?
* How is cumulative revenue changing over time?
* How many users had signed up by a particular date?
* What percentage of total revenue had been collected at each point?

Moving averages help smooth short-term fluctuations and make recent trends easier to observe.

---

## 3. Understanding Window Frames

A **window frame** defines the subset of rows used for a window function's calculation for each current row.

For example:

```sql
SUM(amount_cents) OVER (
    ORDER BY paid_at
)
```

When an aggregate window function uses `ORDER BY` without an explicit frame, PostgreSQL's default frame includes rows from the beginning of the partition through the current row's last peer. When the ordering values are unique, this produces a running total.

### How a Running Total Grows

Suppose the payments are:

| Date       | Amount |
| ---------- | -----: |
| 2024-04-01 |   3997 |
| 2024-04-03 |   8999 |
| 2024-04-12 |   6499 |

The running total is calculated as follows:

| Date       | Amount | Running Total |
| ---------- | -----: | ------------: |
| 2024-04-01 |   3997 |          3997 |
| 2024-04-03 |   8999 |         12996 |
| 2024-04-12 |   6499 |         19495 |

The frame expands as PostgreSQL processes each row, accumulating the values from the beginning through the current row.

---

## 4. Running Total with SUM()

### Syntax

```sql
SELECT
    columns,
    SUM(value_column) OVER (
        ORDER BY sort_column
    ) AS running_total
FROM table_name;
```

### Example: Calculate Cumulative Payment Revenue

```sql
SELECT
    paid_at,
    amount_cents,
    SUM(amount_cents) OVER (
        ORDER BY paid_at
    ) AS running_total_cents
FROM payments
ORDER BY paid_at;
```

### Expected Output

| paid_at                | amount_cents | running_total_cents |
| ---------------------- | -----------: | ------------------: |
| 2024-04-01 10:05:00+09 |         3997 |                3997 |
| 2024-04-03 11:35:00+09 |         8999 |               12996 |
| 2024-04-12 14:05:00+09 |         6499 |               19495 |

**Key observation:** The final running total is `19495` cents, matching the sum of the three payments.

---

## 5. Running Totals with PARTITION BY

`PARTITION BY` allows a running total to restart independently for each group.

### Syntax

```sql
SUM(value_column) OVER (
    PARTITION BY group_column
    ORDER BY sort_column
)
```

### Example: Running Revenue Per Customer

```sql
SELECT
    user_id,
    paid_at,
    amount_cents,
    SUM(amount_cents) OVER (
        PARTITION BY user_id
        ORDER BY paid_at
    ) AS customer_running_total
FROM payments
ORDER BY
    user_id,
    paid_at;
```

### How It Works

* `PARTITION BY user_id` creates a separate window for each customer.
* `ORDER BY paid_at` sorts payments chronologically within each customer.
* `SUM()` accumulates payment amounts separately for each customer.

Each customer's running total starts from their first payment.

---

## 6. Moving Average with AVG()

A moving average calculates the average of values within a defined frame.

The `ROWS BETWEEN` clause controls how many rows are included in the calculation.

### Syntax

```sql
AVG(value_column) OVER (
    ORDER BY sort_column
    ROWS BETWEEN n PRECEDING AND CURRENT ROW
)
```

Here:

* `n PRECEDING` specifies how many preceding rows to include.
* `CURRENT ROW` includes the current row.
* `ROWS` defines the frame by physical row positions.

### Example: Two-Row Moving Average

Calculate the average of the current payment and the immediately preceding payment.

```sql
SELECT
    paid_at,
    amount_cents,
    ROUND(
        AVG(amount_cents) OVER (
            ORDER BY paid_at
            ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
        ),
        2
    ) AS moving_avg_2
FROM payments
ORDER BY paid_at;
```

### Expected Output

| paid_at                | amount_cents | moving_avg_2 |
| ---------------------- | -----------: | -----------: |
| 2024-04-01 10:05:00+09 |         3997 |      3997.00 |
| 2024-04-03 11:35:00+09 |         8999 |      6498.00 |
| 2024-04-12 14:05:00+09 |         6499 |      7749.00 |

### Understanding the Frame

| Row | Included Values | Moving Average |
| --- | --------------- | -------------: |
| 1   | 3997            |        3997.00 |
| 2   | 3997, 8999      |        6498.00 |
| 3   | 8999, 6499      |        7749.00 |

The first row has no preceding row, so its frame contains only itself. From the second row onward, the frame contains at most two rows.

---

## 7. Basic Example: Running Count of User Signups

Window functions can also be used with `COUNT()` to calculate a running count.

### Example

```sql
SELECT
    created_at,
    full_name,
    COUNT(*) OVER (
        ORDER BY created_at
    ) AS signups_so_far
FROM users
ORDER BY created_at;
```

### Expected Output

| created_at             | full_name      | signups_so_far |
| ---------------------- | -------------- | -------------: |
| 2024-01-15 09:12:00+09 | Tanvir Ahmed   |              1 |
| 2024-01-18 14:30:00+09 | Mitsuki Sato   |              2 |
| 2024-02-02 08:05:00+09 | Farzana Rahman |              3 |
| ...                    | ...            |            ... |

**Key observation:** `COUNT(*)` accumulates the number of rows in the ordered window, showing how many users had signed up up to each row.

---

## 8. Real-World Example: Revenue Growth and Percentage of Total

### Business Scenario

The leadership team wants to understand how revenue accumulates over time and what percentage of the final revenue total had been collected at each payment date.

We can calculate the running total first, then use a CTE to calculate the percentage of total revenue.

### SQL Query

```sql
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
FROM running_revenue
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
2. The outer query retrieves the running total for each payment.
3. The scalar subquery finds the final running total using `MAX()`.
4. The percentage calculation shows how much of the final revenue had accumulated at each point.

**Key observation:** The final row reaches `100.0%` because the cumulative revenue equals the final total.

---

## 9. Key Concepts Learned

* A running total accumulates values from the beginning of a window to the current row.
* A moving average calculates the average over a limited window frame.
* `SUM()`, `AVG()`, and `COUNT()` can be used as window functions.
* `ORDER BY` determines the sequence in which the running calculation progresses.
* `PARTITION BY` allows calculations to restart for each group.
* `ROWS BETWEEN` defines the frame boundaries using row positions.
* CTEs can store intermediate window-function results for use in later calculations.
* Running totals can be used to calculate cumulative revenue and progress toward a final total.

---

## 10. Session Summary

| Concept                     | Purpose                                              |
| --------------------------- | ---------------------------------------------------- |
| `SUM() OVER (ORDER BY ...)` | Calculates a running total                           |
| `AVG() OVER (...)`          | Calculates an average over a window                  |
| `COUNT() OVER (...)`        | Calculates a running count                           |
| `PARTITION BY`              | Restarts calculations for each group                 |
| `ROWS BETWEEN`              | Defines the window frame                             |
| CTE + Window Function       | Reuses calculated running totals in further analysis |

### Key Takeaway

**Window frames control which rows contribute to each calculation. With `SUM()`, they create running totals; with `AVG()`, they create moving averages.**

---

