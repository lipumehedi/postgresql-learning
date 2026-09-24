# 📘 Session 04 — CREATE TABLE & Data Types

> **Build the table for real.**
> Learn how to create PostgreSQL tables, choose appropriate data types, apply constraints, and inspect table structure using `psql`.

---

## 📌 Session Information

| Item             | Details                                                   |
| ---------------- | --------------------------------------------------------- |
| **Module**       | Building Blocks                                           |
| **Session**      | 04 / 08                                                   |
| **Level**        | Absolute Beginner                                         |
| **Prerequisite** | Session 03 — Environment & `psql`                         |
| **Database**     | PostgreSQL                                                |
| **Focus**        | `CREATE TABLE`, data types, constraints & table structure |

---

# 🎯 Learning Objectives

By the end of this session, you should be able to:

* Create a PostgreSQL table using `CREATE TABLE`
* Understand columns and data types
* Choose appropriate data types for common use cases
* Apply `PRIMARY KEY` and `NOT NULL`
* Use default values
* Inspect a table with `\d`
* Understand `DROP TABLE`
* Understand `ALTER TABLE`
* Create and populate the course `users` table

---

# 1. 🏗️ What Is `CREATE TABLE`?

`CREATE TABLE` is the SQL statement used to create a new table.

A table definition specifies:

```text
Table Name
     ↓
Columns
     ↓
Data Types
     ↓
Constraints
     ↓
Default Values
```

For example:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

This creates an empty table with a defined structure.

---

# 2. 🧩 What Is a Data Type?

A **data type** determines what kind of value a column can store.

Examples:

```text
INTEGER      → Whole numbers
VARCHAR      → Short text
TEXT         → Long text
BOOLEAN      → true / false
DATE         → Calendar date
TIMESTAMPTZ  → Date + time with time-zone awareness
UUID         → Unique identifier
NUMERIC      → Exact decimal values
```

PostgreSQL checks inserted values against the column's data type.

For example, if a column is defined as:

```sql
created_at TIMESTAMPTZ
```

PostgreSQL expects a timestamp-compatible value rather than arbitrary text such as:

```text
"yesterday"
```

Choosing appropriate data types helps prevent invalid data from entering the database.

---

# 3. 🔬 Anatomy of `CREATE TABLE`

Consider:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Each column definition generally follows this structure:

```text
column_name
     ↓
data_type
     ↓
constraints
     ↓
default value
```

For example:

```text
full_name
   ↓
VARCHAR(100)
   ↓
NOT NULL
```

---

# 4. 🛡️ Constraints

Constraints are rules PostgreSQL enforces on data.

## `PRIMARY KEY`

```sql
id SERIAL PRIMARY KEY
```

The primary key uniquely identifies each row.

---

## `NOT NULL`

```sql
full_name VARCHAR(100) NOT NULL
```

The column cannot contain `NULL`.

---

## `DEFAULT`

```sql
created_at TIMESTAMPTZ DEFAULT now()
```

If a value isn't supplied, PostgreSQL uses:

```sql
now()
```

as the default.

---

# 5. 📊 Core PostgreSQL Data Types

| Data Type       | Stores                                | Common Use                                            |
| --------------- | ------------------------------------- | ----------------------------------------------------- |
| `INTEGER`       | Whole numbers                         | Quantity, age, numeric IDs, foreign keys              |
| `SERIAL`        | Auto-incrementing integer             | Simple generated IDs                                  |
| `NUMERIC(p, s)` | Exact decimal values                  | Money and precise calculations                        |
| `VARCHAR(n)`    | Text up to a specified length         | Names, emails, status values                          |
| `TEXT`          | Variable-length text                  | Descriptions, comments, bios                          |
| `BOOLEAN`       | `true`, `false`, or `NULL`            | Flags such as `is_active`                             |
| `DATE`          | Calendar date                         | Birthday, due date                                    |
| `TIMESTAMP`     | Date and time without time zone       | Situations where timezone is intentionally irrelevant |
| `TIMESTAMPTZ`   | Date and time with time-zone handling | Event timestamps such as `created_at`                 |
| `UUID`          | 128-bit identifier                    | Globally unique or less-guessable identifiers         |

---

# 6. 💰 Choosing the Right Data Type

Data types should match the meaning of the data.

### Whole Number

```sql
quantity INTEGER
```

Good for:

```text
5
100
2500
```

---

### Money

Use:

```sql
price NUMERIC(10,2)
```

rather than floating-point types when exact decimal arithmetic is required.

Example:

```text
1999.99
250.00
```

---

### Short Text

```sql
full_name VARCHAR(100)
```

Useful when the data has a natural maximum length.

---

### Long Text

```sql
description TEXT
```

Useful when there is no meaningful short maximum.

---

### Yes / No

```sql
is_verified BOOLEAN
```

Example values:

```text
true
false
```

---

### Calendar Date

```sql
birth_date DATE
```

Example:

```text
1995-08-21
```

---

### Event Timestamp

```sql
created_at TIMESTAMPTZ
```

Useful for recording when something happened.

---

# 7. 🕒 `TIMESTAMP` vs `TIMESTAMPTZ`

Two commonly encountered timestamp types are:

```text
TIMESTAMP
TIMESTAMPTZ
```

### `TIMESTAMP`

Stores date + time without time-zone information.

### `TIMESTAMPTZ`

PostgreSQL's time-zone-aware timestamp type.

For application events such as:

```text
created_at
updated_at
logged_in_at
payment_at
```

`TIMESTAMPTZ` is often a good choice when the application needs consistent handling of moments in time across time zones.

For this course:

```sql
created_at TIMESTAMPTZ
```

is used for the `users` table.

---

# 8. 💻 Core Table Commands

## Create a Table

```sql
CREATE TABLE table_name (
    column_name data_type constraints
);
```

---

## Drop a Table

```sql
DROP TABLE table_name;
```

⚠️ `DROP TABLE` removes the table and its data.

Use it carefully.

---

## Add a Column

```sql
ALTER TABLE table_name
ADD COLUMN column_name data_type;
```

---

## Remove a Column

```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

`ALTER TABLE` allows you to modify an existing table structure without recreating the entire table.

---

# 9. 🧑‍💻 Creating the `users` Table

This is the main table used throughout Phase 1.

```sql
CREATE TABLE users (
    id            SERIAL        PRIMARY KEY,
    full_name     VARCHAR(100)  NOT NULL,
    email         VARCHAR(255)  NOT NULL,
    created_at    TIMESTAMPTZ   NOT NULL DEFAULT now()
);
```

### Table Structure

```text
users
│
├── id
│    └── SERIAL + PRIMARY KEY
│
├── full_name
│    └── VARCHAR(100) + NOT NULL
│
├── email
│    └── VARCHAR(255) + NOT NULL
│
└── created_at
     └── TIMESTAMPTZ + NOT NULL + DEFAULT now()
```

---

# 10. 🔍 Inspecting the Table with `psql`

After creating the table, run:

```text
\d users
```

You should see a structure similar to:

```text
Table "public.users"

   Column   |           Type           | Nullable |              Default
------------+--------------------------+----------+-----------------------------------
 id         | integer                  | not null | nextval('users_id_seq'::regclass)
 full_name  | character varying(100)   | not null |
 email      | character varying(255)   | not null |
 created_at | timestamp with time zone | not null | now()

Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
```

### What Does This Tell Us?

`users` contains:

```text
id
full_name
email
created_at
```

The output also confirms:

* Data types
* `NOT NULL` rules
* Default value
* Primary key
* Automatically created primary-key index

---

# 11. 🌱 Loading the Course Seed Data

The remaining Phase 1 sessions use the same initial dataset.

The seed data contains **7 users**.

```sql
INSERT INTO users (full_name, email, created_at) VALUES
    ('Tanvir Ahmed',   'tanvir.ahmed@example.com',   '2024-01-15 09:12:00+09'),
    ('Mitsuki Sato',   'mitsuki.sato@example.com',   '2024-01-18 14:30:00+09'),
    ('Farzana Rahman', 'farzana.rahman@example.com', '2024-02-02 08:05:00+09'),
    ('Kenji Watanabe', 'kenji.watanabe@example.com', '2024-02-10 11:47:00+09'),
    ('Rashed Karim',   'rashed.karim@example.com',   '2024-02-14 19:22:00+09'),
    ('Aiko Tanaka',    'aiko.tanaka@example.com',    '2024-03-01 10:00:00+09'),
    ('Nusrat Jahan',   'nusrat.jahan@example.com',   '2024-03-05 16:40:00+09');
```

This creates:

```text
7 rows
IDs: 1 → 7
```

> The `INSERT` syntax will be studied in detail in **Session 05**.

---

# 12. 📋 Expected Table Data

After loading the seed data:

| id | full_name      | email                                                           | created_at |
| -: | -------------- | --------------------------------------------------------------- | ---------- |
|  1 | Tanvir Ahmed   | [tanvir.ahmed@example.com](mailto:tanvir.ahmed@example.com)     | 2024-01-15 |
|  2 | Mitsuki Sato   | [mitsuki.sato@example.com](mailto:mitsuki.sato@example.com)     | 2024-01-18 |
|  3 | Farzana Rahman | [farzana.rahman@example.com](mailto:farzana.rahman@example.com) | 2024-02-02 |
|  4 | Kenji Watanabe | [kenji.watanabe@example.com](mailto:kenji.watanabe@example.com) | 2024-02-10 |
|  5 | Rashed Karim   | [rashed.karim@example.com](mailto:rashed.karim@example.com)     | 2024-02-14 |
|  6 | Aiko Tanaka    | [aiko.tanaka@example.com](mailto:aiko.tanaka@example.com)       | 2024-03-01 |
|  7 | Nusrat Jahan   | [nusrat.jahan@example.com](mailto:nusrat.jahan@example.com)     | 2024-03-05 |

---

# 13. 🔧 `ALTER TABLE` Example

Suppose the application later needs to know whether a user has verified their email.

We could add:

```sql
ALTER TABLE users
ADD COLUMN is_verified BOOLEAN NOT NULL DEFAULT false;
```

This would add:

```text
is_verified
     ↓
BOOLEAN
     ↓
NOT NULL
     ↓
DEFAULT false
```

### ⚠️ Course Note

Do **not** run this command if you want to keep the Phase 1 table exactly as originally defined.

Later sessions assume the original four-column `users` table.

---

# 14. 🔄 What Happens When Data Is Inserted?

When PostgreSQL receives a new row, it checks the defined structure.

Conceptually:

```text
INSERT data
     ↓
Check data types
     ↓
Check constraints
     ↓
Apply defaults
     ↓
Accept or reject
     ↓
Store data
```

For example:

```text
full_name
   ↓
VARCHAR(100)
   ↓
NOT NULL
```

If the value violates the table's rules, PostgreSQL rejects the statement.

This protection is one of the major advantages of structured database design.

---

# 15. ⚠️ Common Beginner Mistakes

## Mistake 1 — Choosing the Wrong Data Type

For example:

```text
created_at VARCHAR(100)
```

would treat timestamps as arbitrary text.

Prefer a timestamp type when the column represents a real point in time.

---

## Mistake 2 — Using Floating Point for Exact Currency

Avoid using floating-point types when exact decimal arithmetic is required for monetary values.

Prefer:

```sql
NUMERIC(10,2)
```

---

## Mistake 3 — Forgetting `NOT NULL`

If a value is required by the application, consider enforcing that requirement at the database level.

Example:

```sql
email VARCHAR(255) NOT NULL
```

---

## Mistake 4 — Dropping a Table Accidentally

This command:

```sql
DROP TABLE users;
```

removes the table and its data.

Always check what you're about to delete.

---

## Mistake 5 — Not Inspecting the Result

After creating a table, verify it:

```text
\d users
```

Don't rely only on memory or assumptions.

---

# 16. 📝 Practice Exercises

### Exercise 1 — Create the Table

Create the `users` table:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

### Exercise 2 — Inspect the Table

Run:

```text
\d users
```

Identify:

* Columns
* Data types
* Constraints
* Default values
* Primary key index

---

### Exercise 3 — List Tables

Run:

```text
\dt
```

Confirm that `users` exists.

---

### Exercise 4 — Load Seed Data

Insert the seven course users.

Then inspect the table.

---

### Exercise 5 — Think About Data Types

Choose an appropriate data type for:

```text
age
email
is_active
birth_date
product_price
created_at
```

Example answer:

```text
age          → INTEGER
email        → VARCHAR / TEXT
is_active    → BOOLEAN
birth_date   → DATE
product_price→ NUMERIC
created_at   → TIMESTAMPTZ
```

---

# 17. 🧠 Key Takeaways

After completing Session 04, you should understand:

```text
CREATE TABLE
     ↓
Table Structure
     ↓
Columns + Data Types
     ↓
Constraints
     ↓
Default Values
     ↓
Stored Data
```

The most important concepts are:

* `CREATE TABLE` creates a table structure.
* Every column needs an appropriate data type.
* Constraints enforce rules.
* `PRIMARY KEY` uniquely identifies rows.
* `NOT NULL` prevents missing required values.
* `DEFAULT` provides a value when one isn't supplied.
* `\d` helps inspect table structure.
* `ALTER TABLE` modifies an existing table.
* `DROP TABLE` removes a table and its data.
* Good data types help PostgreSQL prevent invalid data.

---

# 🛠️ Technologies & Tools

* PostgreSQL
* SQL
* `psql`
* Data Types
* Table Design
* Constraints
* Primary Keys
* Database Schema
* Git & GitHub

---
