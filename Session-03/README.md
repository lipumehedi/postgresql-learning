# 📘 Session 03 — Environment & `psql`

> **Getting PostgreSQL installed, running, and connected.**
> Learn `psql`, PostgreSQL's command-line client, and prepare the database environment used throughout this course.

---

## 📌 Session Information

| Item             | Details                                               |
| ---------------- | ----------------------------------------------------- |
| **Module**       | Foundations                                           |
| **Session**      | 03 / 08                                               |
| **Level**        | Absolute Beginner                                     |
| **Prerequisite** | Session 02 — Core Terminology                         |
| **Database**     | PostgreSQL                                            |
| **Client**       | `psql`                                                |
| **Focus**        | Installation, connection, databases & `psql` commands |

---

# 🎯 Learning Objectives

By the end of this session, you should be able to:

* Understand how PostgreSQL runs on a computer
* Understand what `psql` is
* Connect to a PostgreSQL server
* Understand the difference between SQL and `psql` commands
* Create a PostgreSQL database
* Switch between databases
* List available databases
* List tables
* Inspect table structures
* Exit `psql`
* Recognize and troubleshoot common connection errors

---

# 1. 🐘 What Is PostgreSQL and `psql`?

Before writing SQL, PostgreSQL needs to be installed and running.

PostgreSQL normally runs as a **long-running server process** in the background.

```text
Your Computer
│
├── PostgreSQL Server
│      │
│      └── Database
│
└── psql
       │
       └── Connects to PostgreSQL
```

---

## `psql`

`psql` is PostgreSQL's official **command-line client**.

It allows you to:

* Connect to PostgreSQL
* Execute SQL statements
* Create databases
* Inspect databases and tables
* View query results
* Manage your PostgreSQL session

For this course, `psql` will be the primary tool used to interact with PostgreSQL.

---

## 🖥️ GUI Tools vs `psql`

PostgreSQL also has graphical tools such as **pgAdmin**.

However, learning `psql` first helps you understand what is happening underneath the GUI.

```text
GUI Tool
   ↓
PostgreSQL

psql
   ↓
PostgreSQL
```

Both communicate with the PostgreSQL server.

---

# 2. 💡 Why Do We Need This?

## You Need a Server Before You Need SQL

Sessions 04–08 assume that PostgreSQL is:

```text
✅ Installed
✅ Running
✅ Accessible
✅ Ready to accept connections
```

If the PostgreSQL server is not running, SQL queries cannot be executed.

This is why environment setup comes before learning more SQL syntax.

---

## 👨‍💻 Real-World Backend Use

Backend engineers use `psql` for tasks such as:

* Inspecting database information
* Debugging data problems
* Checking tables
* Running one-off queries
* Investigating production issues
* Verifying database changes

So `psql` is not only a learning tool.

---

# 3. ⚙️ From Installation to Connection

The basic workflow is:

```text
1. Install PostgreSQL
        ↓
2. PostgreSQL Server starts
        ↓
3. Run psql
        ↓
4. Connect to PostgreSQL
        ↓
5. Execute SQL / psql commands
```

---

## 🔌 What Does a Connection Need?

A PostgreSQL connection generally involves:

```text
Host
Port
Database
Username
Password
```

### Default Port

PostgreSQL normally uses:

```text
5432
```

Think of the port as the "door" through which the client connects to the server.

---

## Connection Example

```text
Host:     localhost
Port:     5432
Database: postgres
User:     postgres
Password: ********
```

If any required connection information is incorrect, the connection may fail.

---

# 4. 🔐 Default PostgreSQL Setup

A PostgreSQL installation normally includes a starter database named:

```text
postgres
```

A default administrative role is also commonly created:

```text
postgres
```

Your first connection will often use this role while setting up the environment.

> **Security note:** In real applications, developers normally use dedicated roles with only the permissions they need instead of using the `postgres` superuser for everyday application access.

---

# 5. 💻 Connecting to PostgreSQL

From your terminal:

```bash
psql -U postgres -d postgres
```

This means:

```text
-U postgres
    ↓
Username = postgres

-d postgres
    ↓
Database = postgres
```

---

## 🖥️ Platform-Specific Commands

The first connection can differ depending on your operating system.

| Platform             | Example Command                |
| -------------------- | ------------------------------ |
| **Windows**          | `psql -U postgres -d postgres` |
| **macOS / Homebrew** | `psql postgres`                |
| **Linux / apt**      | `sudo -u postgres psql`        |

The exact authentication setup depends on how PostgreSQL was installed and configured.

---

# 6. 🟢 Recognizing a Successful Connection

After connecting successfully, you may see:

```text
psql (16.4)
Type "help" for help.

postgres=#
```

The prompt:

```text
postgres=#
```

means `psql` is ready to receive commands.

After switching to another database:

```text
codebridge_db=#
```

the prompt changes to reflect the current database.

---

# 7. 🧩 SQL vs `psql` Meta-Commands

Inside `psql`, you will use two different types of commands.

## SQL Statements

SQL statements communicate with PostgreSQL's database system.

Example:

```sql
SELECT 1;
```

SQL statements normally end with:

```text
;
```

---

## `psql` Meta-Commands

Meta-commands are commands provided by `psql` itself.

They begin with:

```text
\
```

They do **not** require a semicolon.

Examples:

```text
\l
\dt
\d users
\q
```

### Easy Rule

```text
SQL statement
    ↓
Ends with ;

psql meta-command
    ↓
Starts with \
    ↓
No ;
```

---

# 8. 🛠️ Essential `psql` Commands

| Command            | Purpose                            |
| ------------------ | ---------------------------------- |
| `\l`               | List all databases                 |
| `\c database_name` | Connect/switch to a database       |
| `\dt`              | List tables                        |
| `\d table_name`    | Describe a table                   |
| `\q`               | Quit `psql`                        |
| `\?`               | Show help for `psql` meta-commands |

---

## `\l` — List Databases

```text
\l
```

Shows databases available on the PostgreSQL server.

---

## `\c` — Connect to a Database

```text
\c codebridge_db
```

Switches the current session to:

```text
codebridge_db
```

---

## `\dt` — List Tables

```text
\dt
```

Shows tables in the current database/schema.

If you have not created any tables yet, you may see:

```text
Did not find any relations.
```

That is normal.

---

## `\d` — Describe a Table

```text
\d users
```

Displays information about a table, including:

* Columns
* Data types
* Constraints
* Other table information

---

## `\q` — Quit

```text
\q
```

Exits `psql` and returns to your normal terminal.

---

## `\?` — Help

```text
\?
```

Displays help for available `psql` meta-commands.

---

# 9. 🗄️ Creating the Course Database

Database creation is done using SQL.

Inside `psql`:

```sql
CREATE DATABASE codebridge_db;
```

If successful:

```text
CREATE DATABASE
```

Then connect to it:

```text
\c codebridge_db
```

You should see something similar to:

```text
You are now connected to database "codebridge_db".
```

---

# 10. 🧪 First Terminal Session

A basic PostgreSQL session may look like this:

```text
$ psql -U postgres -d postgres

psql (16.4)
Type "help" for help.

postgres=# CREATE DATABASE codebridge_db;
CREATE DATABASE

postgres=# \l

                       List of databases
     Name       |  Owner   | Encoding
----------------+----------+----------
 codebridge_db  | postgres | UTF8
 postgres       | postgres | UTF8

postgres=# \c codebridge_db

You are now connected to database "codebridge_db" as user "postgres".

codebridge_db=# \dt

Did not find any relations.
```

### Why does `\dt` show nothing?

Because we have not created any tables yet.

That is exactly what **Session 04** will cover.

---

# 11. 🚀 Setting Up the Course Database

From this point forward, the course uses:

```text
Database: codebridge_db
```

Create it once:

```sql
CREATE DATABASE codebridge_db;
```

Then connect:

```text
\c codebridge_db
```

Verify:

```text
\dt
```

At this point:

```text
PostgreSQL Server
       │
       └── codebridge_db
              │
              └── No tables yet
```

The tables will be created in the next session.

---

# 12. ⚠️ Common Beginner Errors

## Error 1 — Connection Refused

Example:

```text
psql: error: connection to server at "localhost",
port 5432 failed:
Connection refused
Is the server running on that host and accepting TCP/IP connections?
```

### Meaning

The PostgreSQL server is not accepting connections.

### What to check

```text
1. Is PostgreSQL installed?
2. Is the PostgreSQL service running?
3. Is it listening on the expected port?
```

Start the PostgreSQL service using the method appropriate for your operating system.

---

# 13. 🔑 Password Authentication Failed

Example:

```text
FATAL: password authentication failed
for user "postgres"
```

### Meaning

The server is reachable, but the supplied password does not match the configured password.

Remember:

```text
Passwords are case-sensitive.
```

Double-check the password used during PostgreSQL installation.

---

# 14. 🗄️ Database Does Not Exist

Example:

```text
FATAL: database "codebridge_db" does not exist
```

### Meaning

You tried to connect to a database that has not been created yet.

### Solution

First connect to the default database:

```bash
psql -U postgres -d postgres
```

Then:

```sql
CREATE DATABASE codebridge_db;
```

Then:

```text
\c codebridge_db
```

---

# 15. ❗ Forgetting the Semicolon

Suppose you type:

```sql
SELECT 1
```

and the prompt changes to:

```text
codebridge_db-#
```

The `-#` prompt means `psql` is waiting for the rest of the SQL statement.

Usually, you forgot:

```text
;
```

Finish the statement:

```sql
SELECT 1;
```

---

## 🧠 Remember

```text
codebridge_db=#
```

➡️ Ready for a new command.

```text
codebridge_db-#
```

➡️ PostgreSQL is waiting for the rest of the current SQL statement.

---

# 16. 🌐 Host vs Port

A common beginner mistake is confusing the host and port.

### Host

The host identifies **where the PostgreSQL server is running**.

Example:

```text
localhost
```

### Port

The port identifies **which network endpoint PostgreSQL is listening on**.

Default:

```text
5432
```

For a local setup:

```text
localhost:5432
```

means:

> Connect to PostgreSQL running on this computer through port 5432.

---

# 17. 🧠 Complete Connection Flow

Keep this picture in mind:

```text
┌──────────────────┐
│     Terminal     │
│                  │
│      psql        │
└────────┬─────────┘
         │
         │ Connection
         │
         ▼
┌──────────────────┐
│ PostgreSQL Server│
│                  │
│ Port: 5432       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ codebridge_db    │
│                  │
│   public schema  │
│                  │
│   (no tables yet)│
└──────────────────┘
```

---

# 18. 📝 Practice Exercises

Complete these exercises in your own PostgreSQL environment.

### Exercise 1 — Connect

Connect to PostgreSQL using your platform's appropriate command.

---

### Exercise 2 — List Databases

Run:

```text
\l
```

Identify the databases available on your server.

---

### Exercise 3 — Create the Course Database

Run:

```sql
CREATE DATABASE codebridge_db;
```

---

### Exercise 4 — Connect to the Database

Run:

```text
\c codebridge_db
```

---

### Exercise 5 — Check Tables

Run:

```text
\dt
```

Expected result:

```text
Did not find any relations.
```

This is expected because Session 04 has not created any tables yet.

---

### Exercise 6 — Test SQL

Run:

```sql
SELECT 1;
```

Expected result:

```text
 ?column?
----------
        1
```

---

### Exercise 7 — Get Help

Run:

```text
\?
```

Explore some of the available `psql` commands.

---

### Exercise 8 — Exit

Run:

```text
\q
```

You should return to your normal terminal.

---

# 19. 🏆 Best Practices

### 1. Use `\dt` regularly

When learning, frequently check:

```text
\dt
```

This helps you confirm which tables actually exist.

---

### 2. Use `\d table_name`

When working with a table:

```text
\d users
```

This is one of the quickest ways to inspect its structure.

---

### 3. Use the Up Arrow

Instead of retyping a long SQL query, press:

```text
↑ Up Arrow
```

to recall your previous command.

---

### 4. Keep a Dedicated `psql` Terminal

While learning, keeping one terminal dedicated to PostgreSQL can make your workflow easier to follow.

---

### 5. Check the Prompt

Always pay attention to:

```text
database=#
```

versus:

```text
database-#
```

The second usually means `psql` is waiting for you to finish a statement.

---

# 20. 🧠 Key Takeaways

After completing Session 03, you should understand:

```text
PostgreSQL
    ↓
Server
    ↓
psql Client
    ↓
Connection
    ↓
Database
    ↓
Schema
    ↓
Tables
```

And you should know the difference between:

```text
SQL
│
├── SELECT 1;
├── CREATE DATABASE ...;
└── INSERT INTO ...;

psql Commands
│
├── \l
├── \c
├── \dt
├── \d
└── \q
```

### Most Important Commands

```text
\l                  → List databases
\c database_name    → Connect to database
\dt                 → List tables
\d table_name       → Describe table
\q                  → Quit psql
\?                  → Show help
```

---

# 🛠️ Technologies & Tools

* PostgreSQL
* `psql`
* SQL
* Relational Databases
* Command Line
* Terminal
* Database Administration Basics
* Backend Development

---
