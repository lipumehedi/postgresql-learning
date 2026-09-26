# Session 07 — Role and Permission Basics

**CodeBridge Japan | PostgreSQL Learning Journey**

> Giving the application its own limited identity in PostgreSQL instead of connecting as the all-powerful superuser.

---

## 📋 Session Overview

| Category         | Details                                      |
| ---------------- | -------------------------------------------- |
| **Phase**        | Phase 3 — Advanced Queries and Real Patterns |
| **Session**      | 07 / 08                                      |
| **Topic**        | Role and Permission Basics                   |
| **Level**        | Beginner — Intermediate                      |
| **Prerequisite** | Session 06 — Date and Time Details           |
| **Database**     | PostgreSQL                                   |

---

## 📖 1. Where the Story Left Off

CodeBridge is preparing to hand over its codebase to a second backend developer.

During onboarding, Mitu performs a security review and notices that all database connections have been using the default `postgres` superuser.

Mitu explains to Rafi:

> "That was fine for learning, but that's not the kind of thing that should go into production."

The team decides to create a dedicated application role with only the permissions needed for everyday operations.

This session explores how PostgreSQL roles and permissions help protect databases from accidental or unauthorized operations.

---

## 🎯 2. Learning Objectives

By the end of this session, I learned how to:

* Understand PostgreSQL roles and users.
* Create a role that can log in.
* Grant database, schema, and table permissions.
* Use `GRANT` to provide specific privileges.
* Use `REVOKE` to remove permissions.
* Apply the principle of least privilege.
* Create a limited application role.
* Understand the difference between table permissions and ownership.
* Recognize the importance of separate application and migration roles.

---

## 🧠 3. What Is a Role in PostgreSQL?

A **role** is a named identity in PostgreSQL that can own database objects and receive permissions.

A role with the `LOGIN` attribute can connect to the database. This is commonly called a database user.

PostgreSQL uses the same role system for both users and groups.

### Key Points

* A role does not automatically receive access to every database object.
* Permissions must be granted explicitly, subject to PostgreSQL's ownership and privilege rules.
* Roles can be used to manage access for applications, developers, and administrators.
* A role can have permissions to read, insert, update, or delete data without owning the tables.

---

## 🔐 4. Why Are Roles and Permissions Necessary?

### The Problem with Superuser Access

A PostgreSQL superuser has extensive privileges across the database server.

If an application connects as a superuser, a programming error or compromised application could perform destructive operations.

For example, a superuser may be able to:

* Read or modify sensitive data.
* Drop tables and other database objects.
* Change permissions.
* Perform administrative operations.

### The Solution: Principle of Least Privilege

The **principle of least privilege** means granting each role only the permissions it needs to perform its assigned tasks.

| Role             | Typical Access                                            |
| ---------------- | --------------------------------------------------------- |
| `postgres`       | Administrative and superuser privileges                   |
| `codebridge_app` | Only the application permissions explicitly granted to it |

A limited application role reduces the potential damage caused by mistakes or unauthorized actions.

---

## ⚙️ 5. How PostgreSQL Checks Permissions

Before performing an operation, PostgreSQL checks whether the connected role has the required privileges on the relevant database object.

For example:

* `SELECT` requires permission to read data.
* `INSERT` requires permission to add rows.
* `UPDATE` requires permission to modify rows.
* `DELETE` requires permission to remove rows.
* `DROP TABLE` requires ownership or sufficient authority through an applicable administrative role.

A role without the required permissions receives an error instead of being allowed to perform the operation.

---

## 🛠️ 6. CREATE ROLE and GRANT

### 6.1 Create a Role That Can Log In

```sql
CREATE ROLE role_name
WITH LOGIN PASSWORD 'a-strong-password';
```

This creates a role with permission to log in.

**Note:** Use a secure password and avoid storing production credentials directly in source code.

### 6.2 Grant Database Connection Access

```sql
GRANT CONNECT
ON DATABASE database_name
TO role_name;
```

This allows the role to connect to the specified database, subject to other applicable access controls.

### 6.3 Grant Schema Access

```sql
GRANT USAGE
ON SCHEMA schema_name
TO role_name;
```

`USAGE` allows the role to access objects in the schema when it also has the necessary object-level permissions.

### 6.4 Grant Table Permissions

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON table_name
TO role_name;
```

| Permission | Purpose              |
| ---------- | -------------------- |
| `SELECT`   | Read data            |
| `INSERT`   | Add new rows         |
| `UPDATE`   | Modify existing rows |
| `DELETE`   | Delete rows          |

### 6.5 Grant Permissions on All Existing Tables in a Schema

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA schema_name
TO role_name;
```

This applies to existing tables and views covered by the relevant privileges. It does not automatically grant the same permissions on tables created in the future.

### 6.6 Revoke a Permission

```sql
REVOKE permission_name
ON table_name
FROM role_name;
```

Example:

```sql
REVOKE DELETE
ON products
FROM codebridge_app;
```

This removes the role's directly granted `DELETE` privilege on the specified table, provided no other applicable grant or ownership privilege allows it.

---

## 🧪 7. Basic Example — Creating a Limited Role

### Step 1: Create the Application Role

```sql
CREATE ROLE codebridge_app
WITH LOGIN PASSWORD 'change-this-in-production';
```

### Step 2: Grant Database Connection Access

```sql
GRANT CONNECT
ON DATABASE codebridge_db
TO codebridge_app;
```

### Step 3: Grant Schema Access

```sql
GRANT USAGE
ON SCHEMA public
TO codebridge_app;
```

### Step 4: Grant Read Access to the Products Table

```sql
GRANT SELECT
ON TABLE products
TO codebridge_app;
```

### Result

The `codebridge_app` role can connect to `codebridge_db` and read data from `products`, assuming the role has no additional permissions and the table is accessible.

It does not automatically receive access to `users`, `orders`, or other tables.

---

## 💼 8. Real-World Example — Setting Up the Application Role

### Business Requirement

Rafi and Mitu decide that the application needs to:

* Read data from existing tables.
* Insert new rows.
* Update existing rows.
* Delete rows when necessary.
* Avoid creating, dropping, or rebuilding tables.

Database schema changes will be handled separately by an authorized migration role.

### SQL Solution

```sql
CREATE ROLE codebridge_app
WITH LOGIN PASSWORD 'change-this-in-production';

GRANT CONNECT
ON DATABASE codebridge_db
TO codebridge_app;

GRANT USAGE
ON SCHEMA public
TO codebridge_app;

GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA public
TO codebridge_app;
```

**Important:** This example grants privileges on existing tables. Applications using `SERIAL` or identity-related sequences may also need appropriate sequence privileges. Future tables require separately configured permissions, such as `ALTER DEFAULT PRIVILEGES`.

### Step 1: Connect as the Application Role

```bash
psql -U codebridge_app -d codebridge_db
```

### Step 2: Read Data

```sql
SELECT full_name
FROM users
LIMIT 1;
```

**Example output:**

```text
   full_name
----------------
 Tanvir Ahmed
```

This query succeeds if the role has the required `SELECT` permission on `users`.

### Step 3: Attempt a Destructive Operation

```sql
DROP TABLE products;
```

**Expected result:**

```text
ERROR: must be owner of table products
```

The application role cannot drop the table because it was not granted ownership or the authority required to perform that operation.

This demonstrates how a limited role helps protect database structure from accidental destruction.

---

## 📚 9. Key Concepts Learned

| Concept         | Purpose                                                               |
| --------------- | --------------------------------------------------------------------- |
| `CREATE ROLE`   | Creates a new PostgreSQL role                                         |
| `LOGIN`         | Allows a role to connect to PostgreSQL                                |
| `GRANT`         | Gives specific privileges to a role                                   |
| `REVOKE`        | Removes granted privileges                                            |
| `CONNECT`       | Allows connection to a database                                       |
| `USAGE`         | Allows access to a schema's objects when other privileges are present |
| `SELECT`        | Reads table data                                                      |
| `INSERT`        | Adds rows                                                             |
| `UPDATE`        | Modifies rows                                                         |
| `DELETE`        | Removes rows                                                          |
| Least Privilege | Limits access to only what is needed                                  |
| Ownership       | Provides authority over database objects, including schema changes    |

---

## 💡 10. Key Takeaways

* PostgreSQL roles provide a structured way to manage database access.
* A role with `LOGIN` can connect, but it does not automatically receive unrestricted access.
* `GRANT` provides specific permissions, while `REVOKE` removes granted permissions.
* Applications should use dedicated roles rather than superuser accounts.
* Table permissions do not automatically grant ownership or permission to drop tables.
* Existing-table privileges and future-table privileges must be managed separately.
* Separate application and migration roles help keep routine operations distinct from schema administration.

---
