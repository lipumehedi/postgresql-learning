# Session 06 — Date and Time Details

**CodeBridge Japan | PostgreSQL Learning Journey**

> Filling in the gaps that a report doesn't even know it has — using `INTERVAL`, `generate_series()`, and calendar-based monthly reports.

---

## 📋 Session Overview

| Category         | Details                                      |
| ---------------- | -------------------------------------------- |
| **Phase**        | Phase 3 — Advanced Queries and Real Patterns |
| **Session**      | 06 / 08                                      |
| **Topic**        | Date and Time Details                        |
| **Level**        | Beginner — Intermediate                      |
| **Prerequisite** | Session 05 — Set Operations                  |
| **Database**     | PostgreSQL                                   |

---

## 📖 1. Where the Story Left Off

After completing Session 05, Rafi's combined contact list was ready for the marketing campaign.

Leadership then requested a new report:

> "Show us revenue from January to June, month by month, so we can see the first half of the year together."

Rafi used the `revenue_growth` view from Session 04 and grouped payments by month. However, the report showed only April.

Leadership asked:

> "Where did January go? What happened to February and March?"

The problem was not a SQL error. The report simply showed only the months that contained data.

This session explores how to build a complete calendar and display every month, even when no revenue was recorded.

---

## 🎯 2. Learning Objectives

By the end of this session, I learned how to:

* Work with time intervals using `INTERVAL`.
* Generate continuous date and timestamp sequences using `generate_series()`.
* Build a complete calendar using a Common Table Expression (CTE).
* Use `LEFT JOIN` to preserve calendar rows even when no matching data exists.
* Use `COALESCE()` to display zero instead of `NULL`.
* Create monthly revenue reports that include months without transactions.
* Understand why time zones matter in date-based reporting.

---

## 🧠 3. What Are `INTERVAL` and `generate_series()`?

### 3.1 INTERVAL

An `INTERVAL` represents a length of time that PostgreSQL can add to or subtract from a date or timestamp.

**Examples:**

```sql
SELECT
    INTERVAL '1 month' AS one_month,
    INTERVAL '7 days' AS one_week,
    INTERVAL '2 hours' AS two_hours;
```

Intervals can also be used in date calculations:

```sql
SELECT
    DATE '2024-04-15' + INTERVAL '7 days' AS next_week,
    DATE '2024-04-15' - INTERVAL '1 month' AS previous_month;
```

### 3.2 generate_series()

`generate_series()` creates a sequence of values from a starting point to an ending point using a specified step.

**Syntax:**

```sql
SELECT generate_series(
    start_value,
    end_value,
    step_interval
);
```

**Example — Generate monthly timestamps:**

```sql
SELECT generate_series(
    '2024-01-01'::timestamptz,
    '2024-03-01'::timestamptz,
    INTERVAL '1 month'
);
```

**Expected output** *(assuming the session time zone is Asia/Tokyo)*:

```text
2024-01-01 00:00:00+09
2024-02-01 00:00:00+09
2024-03-01 00:00:00+09
```

This produces one row for each month in the specified range.

---

## 🔍 4. Why Is a Complete Calendar Necessary?

### The limitation of GROUP BY

When we group payment records by month, PostgreSQL returns only the months that exist in the data.

For example, if payments exist only in April:

```sql
SELECT
    DATE_TRUNC('month', paid_at) AS month_start,
    SUM(amount_cents) AS total_cents
FROM payments
GROUP BY DATE_TRUNC('month', paid_at)
ORDER BY month_start;
```

The report may return only:

| Month      | Revenue |
| ---------- | ------: |
| April 2024 |   19495 |

January, February, March, May, and June are missing because no matching payment records exist for those months.

**The solution:** Generate a complete calendar independently of the payment data, then use `LEFT JOIN` to attach the actual revenue.

---

## 🗓️ 5. Create a Complete Calendar

### 5.1 Generate a Monthly Calendar

```sql
SELECT generate_series(
    '2024-01-01'::timestamptz,
    '2024-06-01'::timestamptz,
    INTERVAL '1 month'
) AS month_start;
```

This creates six rows, one for each month from January through June.

### 5.2 Generate a Daily Calendar

```sql
SELECT generate_series(
    '2024-04-01'::timestamptz,
    '2024-04-15'::timestamptz,
    INTERVAL '1 day'
) AS day;
```

This creates 15 rows, one for each day from April 1 through April 15.

Daily calendars are useful for daily sales reports, attendance tracking, and other time-based analyses.

---

## 🔗 6. Combine a Calendar with Actual Data

The basic pattern is:

1. Generate a complete calendar using `generate_series()`.
2. Store the calendar in a CTE.
3. Aggregate the actual data by date or month.
4. Use `LEFT JOIN` to attach the aggregated data to the calendar.
5. Use `COALESCE()` to replace missing values with zero.

### General SQL Pattern

```sql
WITH calendar AS (
    SELECT generate_series(
        start_date::timestamptz,
        end_date::timestamptz,
        INTERVAL '1 month'
    ) AS month_start
)
SELECT
    c.month_start,
    COALESCE(SUM(t.value_column), 0) AS total
FROM calendar AS c
LEFT JOIN table_name AS t
    ON DATE_TRUNC('month', t.date_column) = c.month_start
GROUP BY c.month_start
ORDER BY c.month_start;
```

**Important:** This is a general pattern. When joining pre-aggregated monthly data, use the monthly total directly rather than summing it again.

---

## 💼 7. Real-World Example — Monthly Revenue Report

### Business Requirement

Leadership wants a complete monthly revenue report from January to June 2024.

The report must:

* Include all six months.
* Display actual revenue for months with payments.
* Display `0` for months without revenue.
* Keep the months in chronological order.

### SQL Solution

```sql
WITH calendar AS (
    SELECT generate_series(
        '2024-01-01'::timestamptz,
        '2024-06-01'::timestamptz,
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
    ON mr.month_start = c.month_start
ORDER BY c.month_start;
```

### Expected Output

*Assuming the sample data contains payment revenue only in April 2024:*

| month_start            | total_cents |
| ---------------------- | ----------: |
| 2024-01-01 00:00:00+09 |           0 |
| 2024-02-01 00:00:00+09 |           0 |
| 2024-03-01 00:00:00+09 |           0 |
| 2024-04-01 00:00:00+09 |       19495 |
| 2024-05-01 00:00:00+09 |           0 |
| 2024-06-01 00:00:00+09 |           0 |

The report now displays every month, including months without payments.

---

## 🌏 8. Time Zone Considerations

PostgreSQL's `timestamptz` stores an instant in time and displays it according to the session's time zone.

For consistent monthly reports:

* Use a consistent time zone when defining reporting periods.
* Be careful when grouping timestamps near midnight, as the month can differ between time zones.
* Ensure calendar boundaries and payment timestamps are interpreted in the same reporting time zone.

For example, the `+09` offset represents Japan Standard Time (JST).

---

## 📚 9. Key Concepts Learned

| Concept             | Purpose                                            |
| ------------------- | -------------------------------------------------- |
| `INTERVAL`          | Represents a duration of time                      |
| `generate_series()` | Generates a sequence of dates or timestamps        |
| `DATE_TRUNC()`      | Groups timestamps into a common time period        |
| `CTE`               | Organizes a query into named temporary result sets |
| `LEFT JOIN`         | Preserves every row from the calendar              |
| `COALESCE()`        | Replaces `NULL` with a specified value             |
| `ORDER BY`          | Sorts the report chronologically                   |
| `timestamptz`       | Handles timestamps with time zone awareness        |

---

## 💡 10. Key Takeaways

* `GROUP BY` alone cannot generate missing months.
* A calendar generated with `generate_series()` provides a complete time range.
* `LEFT JOIN` keeps all calendar rows, even when no matching data exists.
* `COALESCE()` makes missing revenue values display as zero.
* CTEs help organize complex reporting queries into readable steps.
* Consistent time zone handling is essential for accurate date-based reporting.

---
