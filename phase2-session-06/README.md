# Session 06 — Transactions

**CodeBridge Japan — PostgreSQL Learning**

## Session Overview

| Topic         | Details                           |
| ------------- | --------------------------------- |
| Module        | Relationship and Real Queries     |
| Session       | 06 / 08                           |
| Prerequisites | Session 05 — Subqueries           |
| Level         | Beginner — Intermediate           |
| Main Topic    | Transactions and Data Consistency |

---

## 1. What Is a Transaction?

A **transaction** is a group of SQL statements treated as a single logical unit of work.

The main idea is:

> Either all operations succeed, or none of them are permanently applied.

Think about transferring money between two bank accounts:

* Money is deducted from one account.
* Money is added to another account.

Both operations must succeed together. If one fails, the entire operation should be canceled.

In PostgreSQL, transactions help maintain data consistency when multiple SQL statements depend on one another.

### Three Main Transaction Commands

| Command    | Description                                  |
| ---------- | -------------------------------------------- |
| `BEGIN`    | Starts a transaction.                        |
| `COMMIT`   | Saves all changes made in the transaction.   |
| `ROLLBACK` | Cancels all changes made in the transaction. |

---

## 2. Why Are Transactions Necessary?

### Preventing Incomplete Data

Imagine Rafi is building a **Place New Order** feature.

Placing an order requires two steps:

1. Insert a new row into the `orders` table.
2. Insert the order's items into the `order_items` table.

Without a transaction, the first operation might succeed while the second fails.

This could leave an order without its associated items, creating inconsistent data.

With a transaction, both operations are handled together:

* If both statements succeed, the order is committed.
* If one statement fails, the transaction can be rolled back.

This helps prevent incomplete order records.

---

## 3. Transaction Lifecycle

A transaction begins with `BEGIN` and ends with either `COMMIT` or `ROLLBACK`.

### Successful Transaction

```sql
BEGIN;

-- Execute SQL statements
INSERT INTO categories (name)
VALUES ('Electronics');

COMMIT;
```

The new category is saved permanently after `COMMIT`.

### Canceled Transaction

```sql
BEGIN;

INSERT INTO categories (name)
VALUES ('Temporary Category');

ROLLBACK;
```

The inserted category is canceled, and the database returns to its state before the transaction began.

### Transaction Flow

```text
BEGIN
  |
  v
Execute SQL statements
  |
  v
Are all statements successful?
  |
  +---- Yes ----> COMMIT
  |                |
  |                v
  |          Changes saved
  |
  +---- No -----> ROLLBACK
                   |
                   v
             Changes canceled
```

---

## 4. Basic Transaction Syntax

### Successful Transaction

```sql
BEGIN;

statement_1;
statement_2;
statement_3;

COMMIT;
```

### Transaction with ROLLBACK

```sql
BEGIN;

statement_1;
statement_2;

ROLLBACK;
```

`ROLLBACK` cancels the changes made within the current transaction.

### Updating Multiple Tables

```sql
BEGIN;

UPDATE products
SET stock_qty = stock_qty - 1
WHERE id = 5;

UPDATE orders
SET status = 'completed'
WHERE id = 3;

COMMIT;
```

Both updates are committed together if all statements succeed.

**Important:** This example assumes the product and order IDs exist and the business logic allows these updates. A transaction alone does not validate business rules.

---

## 5. Understanding Transactions in psql

PostgreSQL's `psql` terminal changes its prompt while a transaction is active.

* `codebridge_db=#` — Normal prompt.
* `codebridge_db=*#` — A transaction is in progress.
* `codebridge_db=!#` — The transaction is in an aborted state after an error.

### Example: Using ROLLBACK

```sql
BEGIN;

SELECT stock_qty
FROM products
WHERE id = 5;

UPDATE products
SET stock_qty = 59
WHERE id = 5;

ROLLBACK;

SELECT stock_qty
FROM products
WHERE id = 5;
```

### Expected Behavior

If the original `stock_qty` was `60`:

| Step                            | Stock Quantity |
| ------------------------------- | -------------: |
| Before transaction              |             60 |
| After UPDATE inside transaction |             59 |
| After ROLLBACK                  |             60 |

The update is canceled, so the stock quantity returns to its original value.

---

## 6. Real-World Example — Placing a New Order Safely

Rafi is implementing an order placement feature using the `orders` and `order_items` tables.

The following examples assume the database schema created in Session 01.

### 6.1 Successful Transaction

```sql
BEGIN;

-- Step 1: Create a new order
INSERT INTO orders (user_id, status)
VALUES (4, 'pending');

-- Step 2: Add an item to the order
INSERT INTO order_items (
    order_id,
    product_id,
    quantity,
    unit_price_cents
)
VALUES (
    (
        SELECT id
        FROM orders
        WHERE user_id = 4
          AND status = 'pending'
        ORDER BY id DESC
        LIMIT 1
    ),
    2,
    1,
    999
);

COMMIT;
```

### How It Works

1. `BEGIN` starts the transaction.
2. The first `INSERT` creates a new order.
3. The second `INSERT` adds a product to that order.
4. The subquery retrieves the most recently created pending order for user `4`.
5. `COMMIT` saves both changes.

**Note:** In a production application, using `INSERT ... RETURNING id` to capture the newly created order ID is safer than searching for the latest matching order.

---

### 6.2 Failed Transaction — Invalid Product ID

Now Rafi intentionally uses an invalid `product_id` to demonstrate what happens when a statement fails.

```sql
BEGIN;

-- Step 1: Create a new order
INSERT INTO orders (user_id, status)
VALUES (2, 'pending');

-- Step 2: Attempt to add an invalid product
INSERT INTO order_items (
    order_id,
    product_id,
    quantity,
    unit_price_cents
)
VALUES (
    (
        SELECT id
        FROM orders
        WHERE user_id = 2
          AND status = 'pending'
        ORDER BY id DESC
        LIMIT 1
    ),
    999,
    1,
    999
);
```

### Expected Error

If product ID `999` does not exist in the `products` table, PostgreSQL rejects the insert because of the foreign key constraint.

```text
ERROR: insert or update on table "order_items"
violates foreign key constraint
"order_items_product_id_fkey"

DETAIL: Key (product_id)=(999) is not present
in table "products".
```

### What Happens Next?

After the error, the transaction enters an aborted state.

```sql
ROLLBACK;
```

The `ROLLBACK` cancels the transaction, including the earlier successful insert into `orders`.

**Key takeaway:** The first insert does not remain in the database after the rollback.

---

## 7. PostgreSQL Transaction Behavior

PostgreSQL automatically wraps individual SQL statements in transactions when no explicit transaction block is active.

For example:

```sql
INSERT INTO categories (name)
VALUES ('Electronics');
```

If the statement succeeds, PostgreSQL commits it automatically when using the default autocommit behavior.

When multiple statements need to succeed or fail together, use an explicit transaction:

```sql
BEGIN;

-- Multiple related statements

COMMIT;
```

If a statement fails inside an explicit transaction, PostgreSQL marks the transaction as aborted. Further ordinary statements cannot proceed until the transaction is rolled back.

---

## 8. Practice Exercises

### Exercise 1 — Basic Transaction

1. Start a transaction using `BEGIN`.
2. Insert a new category.
3. Commit the transaction.
4. Verify that the category exists.

### Exercise 2 — Rollback Practice

1. Start a transaction.
2. Update a product's stock quantity.
3. Run `ROLLBACK`.
4. Verify that the stock quantity has returned to its original value.

### Exercise 3 — Multiple Updates

1. Start a transaction.
2. Update a product's stock quantity.
3. Update an order's status.
4. Commit both changes together.

### Exercise 4 — Failed Order Placement

1. Start a transaction.
2. Insert a new order.
3. Try inserting an order item with a nonexistent product ID.
4. Observe the foreign key error.
5. Run `ROLLBACK`.
6. Verify that the newly inserted order does not remain.

---

## 9. Key Concepts Learned

* What a database transaction is.
* Why transactions are important for data consistency.
* How `BEGIN`, `COMMIT`, and `ROLLBACK` work.
* How to update multiple tables within a single transaction.
* How PostgreSQL handles statement failures inside a transaction.
* How foreign key constraints help protect relational data.
* How transactions prevent incomplete order records.
* How PostgreSQL handles individual statements through autocommit.

---

## 10. Key Takeaway

**A transaction makes multiple related SQL statements behave as one logical unit of work.**

Use `BEGIN` to start, `COMMIT` to save successful changes, and `ROLLBACK` to cancel changes when something goes wrong.

Transactions are essential for reliable database operations such as order placement, payments, inventory updates, and account transfers.

---
