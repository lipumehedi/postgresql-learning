# 📘 Session 01 — What Is a Database?

> **Foundations before syntax:** Understand what a database actually is, why backend engineers use databases, and where PostgreSQL fits into a backend system.

---

## 📌 Session Information

| Item             | Details                            |
| ---------------- | ---------------------------------- |
| **Module**       | Foundations                        |
| **Session**      | 01 / 08                            |
| **Level**        | Absolute Beginner                  |
| **Prerequisite** | Python Basics                      |
| **Database**     | PostgreSQL                         |
| **Focus**        | Data, Databases, DBMS, RDBMS & SQL |

---

## 🎯 Learning Objectives

By the end of this session, you should understand:

* What **data** means
* What a **database** is
* What a **DBMS** does
* What an **RDBMS** is
* What **SQL** is used for
* What **PostgreSQL** is
* Why backend applications need databases
* How Python applications communicate with PostgreSQL
* The basic difference between a spreadsheet and a database

---

# 1. 🗄️ What Is a Database?

## Data

**Data** is a collection of facts that can be stored and processed.

Examples:

```text
Name: Tanvir Ahmed
Email: tanvir.ahmed@example.com
Price: 2500
Date: 2024-01-15
```

Data becomes useful when it is properly organized and can be easily searched, updated, and managed.

---

## Database

A **database** is an organized collection of data that allows information to be:

* Stored reliably
* Searched efficiently
* Updated safely
* Retrieved when needed
* Shared between multiple applications or users

For example:

```text
Users
├── ID
├── Full Name
├── Email
└── Created At
```

---

## DBMS — Database Management System

A **DBMS (Database Management System)** is software that manages databases.

It is responsible for things such as:

* Storing data
* Retrieving data
* Updating data
* Protecting data
* Managing multiple connections
* Enforcing data rules

Applications normally communicate with the DBMS instead of directly modifying database files.

---

## RDBMS — Relational Database Management System

An **RDBMS** is a type of DBMS that organizes data using **tables**.

Tables contain:

* **Rows** → individual records
* **Columns** → attributes/fields
* **Relationships** → connections between tables

Examples of relational databases include:

* PostgreSQL
* MySQL
* Microsoft SQL Server
* Oracle Database

---

## SQL — Structured Query Language

**SQL** is the language used to communicate with relational databases.

With SQL, we can:

```text
CREATE   → Create database objects
INSERT   → Add data
SELECT   → Read data
UPDATE   → Modify data
DELETE   → Remove data
```

---

## PostgreSQL

**PostgreSQL** is a powerful, free, open-source **RDBMS**.

It provides:

* SQL support
* Relational tables
* Constraints
* Indexes
* Transactions
* Concurrency control
* Advanced data types
* Powerful query capabilities

In this learning journey, PostgreSQL is the primary database system.

---

# 2. 🔗 How the Terms Connect

Think about the relationship like this:

```text
SQL
 │
 │  Language used to communicate
 ▼
PostgreSQL
 │
 │  RDBMS software
 ▼
Database
 │
 │  Organized collection of data
 ▼
Tables
 │
 ├── Rows
 └── Columns
```

### Simple Explanation

You write:

```sql
SELECT * FROM users;
```

Then:

```text
Python / psql
      │
      │ SQL Query
      ▼
PostgreSQL
      │
      │ Reads the database
      ▼
users table
      │
      ▼
Result rows
```

> **Important:** PostgreSQL is the database engine/software. A database is the actual organized data managed by PostgreSQL.

A single PostgreSQL server can manage multiple databases.

---

# 3. 💡 Why Do We Need Databases?

Imagine CodeBridge Japan stores all student information in:

```text
students.xlsx
```

At first, this may seem convenient.

But backend applications need more than a simple spreadsheet.

| Problem                                  | What a Database Provides             |
| ---------------------------------------- | ------------------------------------ |
| Multiple people edit data simultaneously | **Concurrency control**              |
| Invalid values can be entered            | **Data types & constraints**         |
| Difficult to search large datasets       | **SQL queries & indexes**            |
| File may become corrupted during a save  | **Transactions & reliability**       |
| Data needs relationships                 | **Foreign keys & relational design** |

### Example

Suppose we need:

> "Find all students who joined in March."

With a spreadsheet, this may require manually filtering or searching.

With SQL:

```sql
SELECT * FROM students
WHERE joined_date >= '2024-03-01'
AND joined_date < '2024-04-01';
```

The database can process this query efficiently, even when the dataset becomes very large.

---

# 4. ⚙️ Where PostgreSQL Sits in a Backend System

PostgreSQL normally runs as a separate **database server process**.

A Python backend communicates with PostgreSQL through a database connection.

```text
┌──────────────────────┐
│   Backend Application│
│       Python         │
└──────────┬───────────┘
           │
           │ SQL Query
           ▼
┌──────────────────────┐
│   PostgreSQL Server  │
│                      │
│ • Parses SQL         │
│ • Checks constraints │
│ • Runs queries       │
│ • Manages concurrency│
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│      Database        │
│                      │
│ • Tables             │
│ • Indexes            │
│ • Transactions       │
└──────────────────────┘
```

### Key Idea

Your Python application normally does **not** directly manipulate PostgreSQL's internal database files.

Instead:

```text
Python
  ↓
SQL
  ↓
PostgreSQL
  ↓
Database
  ↓
Result
  ↓
Python
```

---

# 5. 📊 Spreadsheet vs Database

### Spreadsheet

```text
students.xlsx
     │
     └── One file
          └── Data
```

### Database

```text
Application
     │
     ▼
Database Server
     │
     ├── Database 1
     │     ├── Table
     │     └── Table
     │
     ├── Database 2
     │     ├── Table
     │     └── Table
     │
     └── Database 3
```

A database is designed to support multiple applications/users, enforce rules, maintain relationships, and safely manage changing data.

---

# 6. 🗄️ Filing Cabinet Analogy

A simple analogy:

```text
Database
│
├── Table
│    ├── Row
│    ├── Row
│    └── Row
│
└── Table
     ├── Row
     └── Row
```

Think of a database as a **filing cabinet system**.

* **Database** → Entire filing system
* **Table** → One drawer
* **Row** → One folder/record
* **Column** → One specific field
* **Constraint** → Rules about what can be stored

This analogy will become much clearer as we learn table design and relationships.

---

# 7. 💻 First Glimpse of SQL

We will start writing complete SQL statements from later sessions.

For now, recognize these two basic operations.

### Reading Data

```sql
SELECT column_name
FROM table_name;
```

### Adding Data

```sql
INSERT INTO table_name (column_name)
VALUES ('some value');
```

### SQL Statement Terminator

Most SQL statements end with:

```text
;
```

The semicolon indicates that the SQL statement has finished.

---

# 8. 📋 Basic Example — The `users` Table

Throughout Phase 1, we will use a simple `users` table.

Conceptually:

| id | full_name      | email                                                           | created_at |
| -: | -------------- | --------------------------------------------------------------- | ---------- |
|  1 | Tanvir Ahmed   | [tanvir.ahmed@example.com](mailto:tanvir.ahmed@example.com)     | 2024-01-15 |
|  2 | Mitsuki Sato   | [mitsuki.sato@example.com](mailto:mitsuki.sato@example.com)     | 2024-01-18 |
|  3 | Farzana Rahman | [farzana.rahman@example.com](mailto:farzana.rahman@example.com) | 2024-02-02 |

### Understanding the Table

* **Table** → `users`
* **Columns** → `id`, `full_name`, `email`, `created_at`
* **Row** → One complete user record
* **id** → Identifies a user
* **email** → Stores the user's email
* **created_at** → Stores when the user was created

In a later session, we will create this table using SQL.

---

# 9. 🔄 Realistic Example — Signup Flow

Imagine a student creates an account on a CodeBridge Japan website.

### Step 1 — User submits a form

```text
Name: Tanvir Ahmed
Email: tanvir.ahmed@example.com
```

### Step 2 — Python receives the data

```text
Website
   ↓
Python Backend
```

### Step 3 — Backend sends SQL

```text
Python Backend
      ↓
INSERT INTO users ...
      ↓
PostgreSQL
```

### Step 4 — PostgreSQL validates the data

PostgreSQL checks the rules defined for the table.

For example:

```text
Is the required data present?
Is the data type correct?
Does the value violate a constraint?
```

### Step 5 — Data is stored

```text
PostgreSQL
    ↓
users table
    ↓
New user row
```

### Step 6 — Later, the backend retrieves the user

For example:

```sql
SELECT *
FROM users
WHERE email = 'tanvir.ahmed@example.com';
```

PostgreSQL returns the matching record to the Python backend.

---

# 10. 🧠 Key Concepts to Remember

| Concept        | Meaning                                                |
| -------------- | ------------------------------------------------------ |
| **Data**       | Facts or information                                   |
| **Database**   | Organized collection of data                           |
| **DBMS**       | Software that manages databases                        |
| **RDBMS**      | DBMS based on relational tables                        |
| **SQL**        | Language used to communicate with relational databases |
| **PostgreSQL** | Open-source relational database system                 |
| **Table**      | Collection of related records                          |
| **Row**        | One record                                             |
| **Column**     | One attribute/field                                    |

---

# 11. 📝 Practice Questions

Try answering these without looking back at the lesson.

### Beginner

1. What is data?
2. What is a database?
3. What does DBMS mean?
4. What is the difference between DBMS and RDBMS?
5. What is SQL?
6. What is PostgreSQL?

### Understanding

7. Why isn't a spreadsheet always suitable for a backend application?
8. What is a table?
9. What is the difference between a row and a column?
10. How does a Python backend communicate with PostgreSQL?

### Challenge

Explain this flow in your own words:

```text
User
 ↓
Python Backend
 ↓
SQL Query
 ↓
PostgreSQL
 ↓
Database
 ↓
Result
 ↓
Python Backend
 ↓
User
```

---

# 12. ✅ Key Takeaways

After completing Session 01, remember these points:

* A **database** stores organized data.
* A **DBMS** manages databases.
* An **RDBMS** organizes data using related tables.
* **SQL** is used to communicate with relational databases.
* **PostgreSQL** is an open-source RDBMS.
* Tables contain **rows and columns**.
* Backend applications use databases for persistent data storage.
* Python applications communicate with PostgreSQL through SQL/database connections.
* PostgreSQL handles important responsibilities such as data validation, concurrency, and reliable transactions.

---

# 🛠️ Technologies & Tools

* PostgreSQL
* SQL
* Relational Database Concepts
* Python
* Backend Development
* Git
* GitHub

---

## 👨‍💻 Learning Journey

This session is part of my structured **PostgreSQL & SQL learning journey**, focused on building a strong foundation for backend development with Python.

> **Learn → Practice → Build → Document → Improve**
