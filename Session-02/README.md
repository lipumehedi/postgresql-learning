# 📘 Session 02 — Core Terminology

> **Build the vocabulary before writing more SQL.**
> Learn the essential PostgreSQL and relational database terms that will appear throughout the rest of this course.

---

## 📌 Session Information

| Item             | Details                                                         |
| ---------------- | --------------------------------------------------------------- |
| **Module**       | Foundations                                                     |
| **Session**      | 02 / 08                                                         |
| **Level**        | Absolute Beginner                                               |
| **Prerequisite** | Session 01 — What Is a Database?                                |
| **Database**     | PostgreSQL                                                      |
| **Focus**        | Database terminology, relationships, constraints & transactions |

---

## 🎯 Learning Objectives

By the end of this session, you should understand:

* What a **database** is
* What a **schema** is
* What a **table** is
* What a **row** and **column** are
* What a **primary key** does
* What a **foreign key** does
* What a **constraint** is
* What an **index** is
* What a **query** means
* What a **transaction** is
* How these concepts connect inside PostgreSQL

---

# 1. 📚 A Shared Vocabulary

This session is a **glossary**, not a command-focused lesson.

These ten terms will appear repeatedly in:

* PostgreSQL documentation
* SQL queries
* Database design
* Backend development
* PostgreSQL error messages
* Future sessions in this course

Understanding this vocabulary early makes later SQL much easier to follow.

---

## 🔑 The 10 Core Terms

| Term            | Definition                                                                                 |
| --------------- | ------------------------------------------------------------------------------------------ |
| **Database**    | A named, organized collection of data managed by PostgreSQL.                               |
| **Schema**      | A named grouping of database objects such as tables inside a database.                     |
| **Table**       | A structure that stores one kind of data using rows and columns.                           |
| **Row**         | One complete record in a table.                                                            |
| **Column**      | A named attribute or field that describes a type of data.                                  |
| **Primary Key** | A column or group of columns that uniquely identifies each row.                            |
| **Foreign Key** | A column that references a key in another table and creates a relationship between tables. |
| **Constraint**  | A rule enforced by PostgreSQL to control what data can be stored.                          |
| **Index**       | An additional data structure that helps PostgreSQL find rows faster.                       |
| **Query**       | A SQL statement used to read or modify data.                                               |
| **Transaction** | A group of database operations treated as one unit of work.                                |

> **Note:** Although this course refers to the "10 core terms," the glossary contains **11 terms** because both **Database** and **Schema** are essential concepts.

---

# 2. 🧩 Why Precise Terminology Matters

Beginners sometimes use these words interchangeably:

```text
Database
Table
Column
Field
Query
Index
Constraint
```

But they represent different concepts.

For example:

❌ "Add a column to the database."

More precisely:

✅ "Add a column to the `users` table."

Similarly:

❌ "The query only means reading data."

A query can be used to:

```text
SELECT  → Read data
INSERT  → Add data
UPDATE  → Modify data
DELETE  → Remove data
```

### Why this matters

PostgreSQL error messages also use this terminology.

For example:

```text
violates foreign key constraint
```

To understand this error, you need to know:

```text
Foreign Key
      +
Constraint
```

Precise vocabulary helps you understand errors, documentation, and conversations with other developers.

---

# 3. 🏗️ From Database Down to a Single Value

The first five concepts can be understood as a hierarchy:

```text
Database
   │
   └── Schema
         │
         └── Table
               │
               ├── Row
               │
               └── Column
```

### Example

```text
Database: codebridge_db
│
└── Schema: public
      │
      └── Table: users
            │
            ├── id
            ├── full_name
            ├── email
            └── created_at
```

The table contains records:

| id | full_name      | email                                                           |
| -: | -------------- | --------------------------------------------------------------- |
|  1 | Tanvir Ahmed   | [tanvir.ahmed@example.com](mailto:tanvir.ahmed@example.com)     |
|  2 | Mitsuki Sato   | [mitsuki.sato@example.com](mailto:mitsuki.sato@example.com)     |
|  3 | Farzana Rahman | [farzana.rahman@example.com](mailto:farzana.rahman@example.com) |

Here:

* `users` → **Table**
* `id`, `full_name`, `email` → **Columns**
* Each person → **Row**
* `public` → **Schema**
* `codebridge_db` → **Database**

---

# 4. 🔐 Primary Key

A **Primary Key** uniquely identifies each row in a table.

Example:

| id | full_name      |
| -: | -------------- |
|  1 | Tanvir Ahmed   |
|  2 | Mitsuki Sato   |
|  3 | Farzana Rahman |

Here:

```text
id → PRIMARY KEY
```

The primary key must be:

* Unique
* Required
* Stable enough to identify a record

Two rows cannot have the same primary key value.

### Example

```text
id = 1  → Tanvir Ahmed
id = 2  → Mitsuki Sato
id = 3  → Farzana Rahman
```

This allows the backend to identify one exact user.

---

# 5. 🔗 Foreign Key

A **Foreign Key** creates a relationship between two tables.

For example:

```text
users
┌────┬──────────────┐
│ id │ name         │
├────┼──────────────┤
│ 1  │ Tanvir       │
│ 2  │ Mitsuki      │
└────┴──────────────┘
       ▲
       │
       │ referenced by
       │
orders
┌────┬──────────────┐
│ id │ user_id      │
├────┼──────────────┤
│ 101│ 1            │
│ 102│ 2            │
└────┴──────────────┘
```

Here:

```text
users.id
   ▲
   │
orders.user_id
```

`orders.user_id` is a **Foreign Key** referencing `users.id`.

The foreign key helps PostgreSQL ensure that the referenced user actually exists.

---

## ⚠️ Important

Our current `users` table does **not** have a foreign key.

That is because it does not reference another table yet.

Relationships between multiple tables will be introduced in a later phase.

---

# 6. 🛡️ Constraints

A **constraint** is a rule enforced by PostgreSQL.

For example:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL
);
```

Here we have several constraints:

### Primary Key

```sql
PRIMARY KEY
```

Ensures the value uniquely identifies a row.

### NOT NULL

```sql
NOT NULL
```

Prevents a required column from being empty.

### Foreign Key

```sql
REFERENCES users(id)
```

Ensures that a referenced value exists in the related table.

---

# 7. ⚡ Index

An **index** is an additional data structure that helps PostgreSQL find rows efficiently.

Imagine a table containing:

```text
10 rows
100 rows
10,000 rows
1,000,000 rows
```

Searching every row can become expensive.

An index can help PostgreSQL locate matching rows more efficiently.

### Simple analogy

Think of a book.

Without an index:

```text
Search → Read every page
```

With an index:

```text
Search → Find the topic in the index → Go to the page
```

PostgreSQL automatically creates an index for a primary key.

Other columns may also be indexed when appropriate.

> **Important:** An index is not automatically created for every column.

---

# 8. 🔎 Query

A **query** is a SQL statement used to interact with database data.

For example:

```sql
SELECT *
FROM users;
```

This query asks PostgreSQL to return the records from the `users` table.

Queries can also modify data:

```sql
INSERT INTO users (...);
UPDATE users SET ...;
DELETE FROM users WHERE ...;
```

So:

> **Query ≠ only SELECT**

A query can read, insert, update, or delete data.

---

# 9. 🔄 Transaction

A **transaction** groups multiple database operations into one logical unit.

The key idea is:

```text
ALL operations succeed
        OR
ALL operations fail
```

For example, transferring money between two accounts may require:

```text
1. Subtract money from Account A
2. Add money to Account B
```

These operations should normally succeed together.

If the first operation succeeds but the second fails, the database should be able to roll back the transaction.

Conceptually:

```text
BEGIN
   │
   ├── Operation 1
   │
   ├── Operation 2
   │
   └── Operation 3
   │
COMMIT
```

If something goes wrong:

```text
ROLLBACK
```

Transactions will be explored in more detail later.

---

# 10. 🧠 Putting Everything Together

Here is the relationship between the main concepts:

```text
┌─────────────────────────────┐
│          DATABASE           │
│        codebridge_db        │
│                             │
│  ┌───────────────────────┐  │
│  │       SCHEMA          │  │
│  │       public          │  │
│  │                       │  │
│  │  ┌─────────────────┐  │  │
│  │  │     TABLE       │  │  │
│  │  │     users       │  │  │
│  │  │                 │  │  │
│  │  │ id  ← PK        │  │  │
│  │  │ name             │  │  │
│  │  │ email            │  │  │
│  │  │ created_at       │  │  │
│  │  └─────────────────┘  │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

And between tables:

```text
┌──────────────┐             ┌──────────────┐
│    users     │             │    orders    │
├──────────────┤             ├──────────────┤
│ id ← PK      │◄────────────│ user_id ← FK │
│ name         │             │ id ← PK      │
└──────────────┘             └──────────────┘
```

---

# 11. 📋 Our `users` Table Vocabulary

Let's map today's terminology to our running example.

| Term            | Example                                      |
| --------------- | -------------------------------------------- |
| **Database**    | `codebridge_db`                              |
| **Schema**      | `public`                                     |
| **Table**       | `users`                                      |
| **Row**         | One specific user record                     |
| **Column**      | `id`, `full_name`, `email`, `created_at`     |
| **Primary Key** | `id`                                         |
| **Foreign Key** | None yet                                     |
| **Constraint**  | `NOT NULL`, `PRIMARY KEY`                    |
| **Index**       | Automatically created for the primary key    |
| **Query**       | Any SQL statement executed against the table |
| **Transaction** | A group of operations treated as one unit    |

---

# 12. 💬 Real-World Developer Example

A backend engineer might describe their work like this:

> "I added a query that reads from the `users` table. Each row has a unique primary key, so I used the `id` column to fetch one specific user. Since this was a single read operation, I didn't need a multi-step transaction."

Breaking it down:

```text
Query
  ↓
users table
  ↓
Row
  ↓
Primary Key
  ↓
id column
  ↓
Transaction when multiple operations must work together
```

The goal is not simply to memorize definitions.

The goal is to understand what developers mean when they use these words.

---

# 13. ⚠️ Common Beginner Mistakes

## Mistake 1 — Database vs Table

❌ "Add a column to the database."

✅ "Add a column to the `users` table."

A database contains tables; a database itself does not have columns.

---

## Mistake 2 — Field vs Column

In many older systems and conversations, **field** may be used informally to mean **column**.

For this course, use:

```text
Column
```

---

## Mistake 3 — Every Column Has an Index

Not every column automatically gets an index.

For example:

```text
id          → Primary key → index automatically created
email       → No automatic index just because it is a column
```

Indexes should be added based on actual query and performance needs.

---

## Mistake 4 — Primary Key Can Be Empty

A primary key must identify a row uniquely.

PostgreSQL will not allow:

```text
NULL primary key
```

or duplicate primary key values.

---

## Mistake 5 — Query Means SELECT Only

A query can perform different operations:

```text
SELECT
INSERT
UPDATE
DELETE
```

So "query" is a broader term than "read operation."

---

# 14. ✅ Best Practices

### 1. Use precise terminology

Instead of:

```text
"The database is broken."
```

Try:

```text
"The users table has a constraint violation."
```

Precise language makes debugging easier.

---

### 2. Give every table a clear primary key

Think about the table's unique identifier when designing it.

---

### 3. Understand relationships before adding foreign keys

Know which table is the parent and which table references it.

---

### 4. Don't add indexes blindly

Indexes can improve reads but also have storage and write-maintenance costs.

---

### 5. Revisit this glossary

After Sessions 04 and 05, come back to this page.

These definitions will become much easier to understand once you start creating tables and writing real SQL.

---

# 15. 📝 Practice Questions

### Beginner

1. What is a database?
2. What is a schema?
3. What is a table?
4. What is a row?
5. What is a column?
6. What is a primary key?

### Intermediate

7. What is a foreign key?
8. Why are constraints useful?
9. What is an index?
10. Can a query modify data?
11. What is a transaction?

### Challenge

Explain this structure in your own words:

```text
Database
   ↓
Schema
   ↓
Table
   ↓
Rows + Columns
   ↓
Primary Key
   ↓
Relationships through Foreign Keys
   ↓
Constraints + Indexes
   ↓
Queries + Transactions
```

---

# 🧠 Key Takeaways

After completing Session 02, you should be comfortable with:

```text
Database
    ↓
Schema
    ↓
Table
    ↓
Row + Column
    ↓
Primary Key
    ↓
Foreign Key
    ↓
Constraint
    ↓
Index
    ↓
Query
    ↓
Transaction
```

These terms form the foundation for understanding relational databases and PostgreSQL.

---

# 🛠️ Technologies & Concepts

* PostgreSQL
* SQL
* Relational Databases
* Database Design
* Primary Keys
* Foreign Keys
* Constraints
* Indexes
* Queries
* Transactions
* Backend Development
