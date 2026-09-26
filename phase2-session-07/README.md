# Session 07 — Constraint Details (UNIQUE, CHECK, ON DELETE)

**CodeBridge Japan — PostgreSQL Learning**

## Session Overview

| Topic         | Details                                              |
| ------------- | ---------------------------------------------------- |
| Module        | Relationship and Real Queries                        |
| Session       | 07 / 08                                              |
| Prerequisites | Session 06 — Transactions                            |
| Level         | Beginner — Intermediate                              |
| Main Topics   | UNIQUE, CHECK, ON DELETE CASCADE, ON DELETE SET NULL |

---

## 1. What Are Constraints?

**Constraints** are rules enforced by PostgreSQL to maintain data accuracy and consistency.

In earlier sessions, we learned about:

* `NOT NULL` — Prevents a column from storing `NULL`.
* `PRIMARY KEY` — Uniquely identifies each row.
* `FOREIGN KEY` — Maintains relationships between tables.

In this session, we will explore two additional constraints:

| Constraint | Purpose                                                          |
| ---------- | ---------------------------------------------------------------- |
| `UNIQUE`   | Prevents duplicate values in a column or combination of columns. |
| `CHECK`    | Ensures that values satisfy a specified condition.               |

We will also learn how foreign key deletion rules work:

* `ON DELETE CASCADE`
* `ON DELETE SET NULL`

---

## 2. Why Are Constraints Necessary?

### Preventing Invalid Data

Imagine a customer registers twice using the same email address.

If the database only uses `NOT NULL`, it will allow duplicate email addresses.

Similarly, without a `CHECK` constraint, a product could be inserted with a negative price or an order item could have a quantity of zero.

Database constraints protect data even when:

* Application code contains a bug.
* A developer inserts data directly using `psql`.
* Another application accesses the same database.

**Key idea:** Business rules that must always be enforced should be protected at the database level.

---

## 3. How Constraints Work

When PostgreSQL executes an `INSERT` or `UPDATE`, it checks the relevant constraints.

For example, inserting a product may require:

1. The required fields to satisfy `NOT NULL`.
2. The product's unique values to satisfy `UNIQUE`.
3. The price to satisfy the `CHECK` condition.
4. The referenced category to exist if a foreign key is used.

If a constraint is violated, PostgreSQL rejects the statement.

For a multi-row statement, a constraint violation normally prevents that statement from partially applying its changes.

---

## 4. UNIQUE Constraint

The `UNIQUE` constraint prevents duplicate values in a column or combination of columns.

### 4.1 UNIQUE in a New Table

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(100),
    email VARCHAR(150) UNIQUE
);
```

This ensures that non-`NULL` email values are unique.

### 4.2 Adding UNIQUE to an Existing Table

```sql
ALTER TABLE users
ADD CONSTRAINT users_email_unique
UNIQUE (email);
```

This creates a unique constraint on the `email` column in the `users` table.

**Important:** If duplicate non-`NULL` email values already exist, PostgreSQL will reject the constraint addition until those duplicates are resolved.

### 4.3 Testing Duplicate Emails

```sql
INSERT INTO users (full_name, email, created_at)
VALUES (
    'Fake Tanvir',
    'tanvir.ahmed@example.com',
    NOW()
);
```

If the email already exists, PostgreSQL returns an error similar to:

```text
ERROR: duplicate key value violates unique constraint
"users_email_unique"
```

### 4.4 UNIQUE on Multiple Columns

A unique constraint can apply to a combination of columns.

```sql
CREATE TABLE course_enrollments (
    id SERIAL PRIMARY KEY,
    student_id INT,
    course_id INT,
    UNIQUE (student_id, course_id)
);
```

This prevents the same student from enrolling in the same course more than once.

**Note:** PostgreSQL allows multiple `NULL` values in a regular unique constraint by default. Use `NOT NULL` as well if a value must always be provided.

---

## 5. CHECK Constraint

A `CHECK` constraint ensures that a value satisfies a specified condition.

### 5.1 CHECK in a New Table

```sql
CREATE TABLE products_demo (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price_cents INT CHECK (price_cents > 0)
);
```

The price must be greater than zero whenever it is not `NULL`.

### 5.2 Adding CHECK to an Existing Table

```sql
ALTER TABLE products
ADD CONSTRAINT products_price_positive
CHECK (price_cents > 0);
```

This prevents products from having a zero or negative price.

### 5.3 Testing the Constraint

```sql
INSERT INTO products (name, price_cents)
VALUES ('Test Product', -100);
```

If the table has the `products_price_positive` constraint, PostgreSQL rejects the insert.

### Important: CHECK and NULL

A `CHECK` constraint rejects a row only when its condition evaluates to `FALSE`.

If the condition evaluates to `NULL` (unknown), the constraint passes.

To require a value and ensure it is positive, use both:

```sql
price_cents INT NOT NULL CHECK (price_cents > 0)
```

---

## 6. Additional Practical Constraints

### 6.1 Order Item Quantity Must Be Positive

```sql
ALTER TABLE order_items
ADD CONSTRAINT order_items_quantity_positive
CHECK (quantity > 0);
```

This prevents zero or negative quantities.

### 6.2 Product Stock Cannot Be Negative

```sql
ALTER TABLE products
ADD CONSTRAINT products_stock_nonnegative
CHECK (stock_qty >= 0);
```

This ensures that stock quantity cannot be below zero.

### 6.3 Category Names Must Be Unique

```sql
ALTER TABLE categories
ADD CONSTRAINT categories_name_unique
UNIQUE (name);
```

This prevents duplicate non-`NULL` category names.

---

## 7. Understanding ON DELETE Rules

Foreign key constraints define what happens to related rows when a referenced parent row is deleted.

Two important options are:

| Rule                 | Behavior                                           |
| -------------------- | -------------------------------------------------- |
| `ON DELETE CASCADE`  | Deletes related child rows automatically.          |
| `ON DELETE SET NULL` | Sets the referencing foreign key values to `NULL`. |

These options help define how related data should behave when a parent record is removed.

### 7.1 ON DELETE CASCADE

Example:

```sql
FOREIGN KEY (order_id)
REFERENCES orders(id)
ON DELETE CASCADE
```

If an order is deleted, the associated rows in `order_items` are also deleted.

Use this when child records should not exist without their parent.

### 7.2 ON DELETE SET NULL

Example:

```sql
FOREIGN KEY (category_id)
REFERENCES categories(id)
ON DELETE SET NULL
```

If a category is deleted, the associated products remain, but their `category_id` becomes `NULL`.

The referencing column must allow `NULL`.

---

## 8. Changing an Existing Foreign Key

PostgreSQL does not provide a direct command to modify the `ON DELETE` action of an existing foreign key.

The usual process is:

1. Drop the existing foreign key constraint.
2. Add a new foreign key constraint with the desired deletion rule.

### Example: Changing to ON DELETE SET NULL

```sql
ALTER TABLE products
DROP CONSTRAINT products_category_id_fkey;

ALTER TABLE products
ADD CONSTRAINT products_category_id_fkey
FOREIGN KEY (category_id)
REFERENCES categories(id)
ON DELETE SET NULL;
```

This replaces the existing foreign key with one that sets `category_id` to `NULL` when the referenced category is deleted.

**Important:** If you are unsure of the constraint name, inspect the table definition using `psql`:

```sql
\d products
```

Dropping and recreating a constraint should be done carefully, especially in a production database.

---

## 9. Real-World Example — ON DELETE SET NULL

### Scenario

Mitu wants products to remain in the database even when their category is deleted.

For example, deleting a category should not delete the products assigned to it.

Instead, the products should remain with `category_id = NULL`.

### Step 1: Replace the Foreign Key

```sql
ALTER TABLE products
DROP CONSTRAINT products_category_id_fkey;

ALTER TABLE products
ADD CONSTRAINT products_category_id_fkey
FOREIGN KEY (category_id)
REFERENCES categories(id)
ON DELETE SET NULL;
```

### Step 2: Create a Temporary Category

```sql
INSERT INTO categories (name)
VALUES ('Temporary Test Category');
```

### Step 3: Assign a Product to the Temporary Category

```sql
UPDATE products
SET category_id = (
    SELECT id
    FROM categories
    WHERE name = 'Temporary Test Category'
)
WHERE id = 5;
```

### Step 4: Delete the Temporary Category

```sql
DELETE FROM categories
WHERE name = 'Temporary Test Category';
```

### Step 5: Verify the Product

```sql
SELECT name, category_id
FROM products
WHERE id = 5;
```

### Expected Result

If product `5` is named `Desk Lamp`, and the temporary category was successfully deleted:

| name      | category_id |
| --------- | ----------- |
| Desk Lamp | NULL        |

The product remains in the database, but its category reference is cleared.

### Step 6: Restore the Original Category

```sql
UPDATE products
SET category_id = (
    SELECT id
    FROM categories
    WHERE name = 'Gear'
)
WHERE id = 5;
```

This restores the product's category if the `Gear` category exists.

---

## 10. Practice Exercises

### Exercise 1 — UNIQUE Constraint

1. Add a unique constraint to `users.email`.
2. Insert a new user with a unique email.
3. Try inserting another user with the same email.
4. Observe the error.

### Exercise 2 — CHECK Constraint

1. Add a positive price constraint to `products`.
2. Insert a product with a positive price.
3. Try inserting a product with a negative price.
4. Observe the constraint violation.

### Exercise 3 — Quantity Validation

1. Add a `CHECK` constraint to `order_items.quantity`.
2. Try inserting an order item with quantity `0`.
3. Try inserting an order item with quantity `-1`.
4. Confirm that both are rejected.

### Exercise 4 — ON DELETE SET NULL

1. Create a temporary category.
2. Assign a product to that category.
3. Delete the category.
4. Verify that the product remains with `category_id = NULL`.

### Exercise 5 — ON DELETE CASCADE

1. Create a test parent-child relationship with `ON DELETE CASCADE`.
2. Insert a parent row and its child row.
3. Delete the parent row.
4. Verify that the child row is also deleted.

---

## 11. Key Concepts Learned

* What database constraints are and why they matter.
* How `UNIQUE` prevents duplicate values.
* How `CHECK` enforces custom data validation rules.
* How to add constraints using `ALTER TABLE`.
* How to handle existing duplicate data before adding a unique constraint.
* Why `CHECK` alone does not prevent `NULL` values.
* How `ON DELETE CASCADE` removes related child records.
* How `ON DELETE SET NULL` preserves child records while clearing their references.
* How to replace an existing foreign key constraint.
* How database constraints improve data integrity.

---

## 12. Key Takeaway

**Constraints protect the database from invalid and inconsistent data.**

Use `UNIQUE` to prevent duplicate values, `CHECK` to enforce custom conditions, and foreign key deletion rules to control what happens to related records.

These rules make database behavior more predictable and reliable.

---
