# Session 01 — Multi-Table Schemas & Foreign Keys

> **CodeBridge Japan — PostgreSQL Zero to Hero**

Learn how real-world PostgreSQL databases are split into multiple related tables and connected using **foreign keys**.

---

## 📌 Session Overview

| Item             | Details                                            |
| ---------------- | -------------------------------------------------- |
| **Session**      | 01 / 08                                            |
| **Module**       | Relationships & Real Queries                       |
| **Prerequisite** | Phase 1 Complete                                   |
| **Level**        | Beginner, Building Up                              |
| **Main Topics**  | Multi-table schemas · Foreign Keys · Relationships |

---

# 1. What Is It?

## Tables That Point to Each Other

In Phase 1, we worked mainly with:

```text
users
products
```

A real online store needs more information:

* Who placed an order?
* Which products were ordered?
* How many products were ordered?
* How much was paid?
* Which category does a product belong to?

Instead of putting everything into one giant table, we separate the data into multiple tables and connect them using **foreign keys**.

This connection is called a **relationship**.

---

# 2. One-to-Many Relationships

The most common relationship is **one-to-many**.

Example:

```text
One User
   │
   ├── Order 1
   ├── Order 2
   └── Order 3
```

One user can have many orders, but each order belongs to one user.

Instead of storing the user's name and email inside every order, the order stores only the user's ID:

```text
orders.user_id → users.id
```

This reduces duplicate data and makes the database easier to maintain.

---

# 3. Why Do We Need Multiple Tables?

## The Problem With One Big Table

Imagine storing orders like this:

| order_id | user_name    | user_email                                                  | product_name                  | price_cents |
| -------: | ------------ | ----------------------------------------------------------- | ----------------------------- | ----------: |
|        1 | Tanvir Ahmed | [tanvir.ahmed@example.com](mailto:tanvir.ahmed@example.com) | Beginner SQL Handbook         |        1999 |
|        2 | Tanvir Ahmed | [tanvir.ahmed@example.com](mailto:tanvir.ahmed@example.com) | PostgreSQL Cheat Sheet Poster |         999 |

Tanvir's email appears multiple times.

If his email changes, every duplicated copy must be updated.

This creates **data duplication** and can cause inconsistent data.

Instead:

```text
users
├── id
├── full_name
└── email

orders
├── id
└── user_id
```

The user's information is stored once, and orders reference the user through `user_id`.

---

# 4. The Full CodeBridge Store Schema

The store will use several related tables:

```text
users
   │
   └── orders
          │
          └── order_items ─── products
                                │
                                └── categories

orders ─── payments
```

### Relationships

| Table         | Foreign Key   | References       |
| ------------- | ------------- | ---------------- |
| `products`    | `category_id` | `categories(id)` |
| `orders`      | `user_id`     | `users(id)`      |
| `order_items` | `order_id`    | `orders(id)`     |
| `order_items` | `product_id`  | `products(id)`   |
| `payments`    | `order_id`    | `orders(id)`     |

Each arrow means:

> The foreign key value must reference a valid row in the parent table.

---

# 5. What Happens When a Parent Row Is Deleted?

Suppose a user has existing orders.

What should happen if that user is deleted?

PostgreSQL supports several behaviors.

| Option               | Meaning                                     |
| -------------------- | ------------------------------------------- |
| Default              | Block the delete if child rows exist        |
| `ON DELETE CASCADE`  | Delete related child rows automatically     |
| `ON DELETE SET NULL` | Keep child rows but remove the relationship |

Example:

```sql
order_id INTEGER
REFERENCES orders(id)
ON DELETE CASCADE
```

This is useful for payments because a payment has no meaning if its order no longer exists.

---

# 6. Foreign Key Syntax

## Add a Foreign Key to an Existing Table

```sql
ALTER TABLE child_table
ADD COLUMN parent_id INTEGER
REFERENCES parent_table(id);
```

Example:

```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

ALTER TABLE products
ADD COLUMN category_id INTEGER
REFERENCES categories(id);
```

---

## Create a Foreign Key When Creating a Table

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL
        REFERENCES users(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Here:

```text
orders.user_id
      ↓
users.id
```

---

# 7. Building the CodeBridge Store

## Step 1 — Create Categories

```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

INSERT INTO categories (name) VALUES
    ('Books'),
    ('Study Aids'),
    ('Gear');
```

---

## Step 2 — Connect Products to Categories

The `products` table already exists from Phase 1.

Add the foreign key:

```sql
ALTER TABLE products
ADD COLUMN category_id INTEGER
REFERENCES categories(id);
```

Assign categories:

```sql
UPDATE products
SET category_id = 1
WHERE id = 1;

UPDATE products
SET category_id = 2
WHERE id = 2;

UPDATE products
SET category_id = 3
WHERE id IN (3, 4, 5);
```

---

# 8. Create Orders

Each order belongs to one user.

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL
        REFERENCES users(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Insert sample orders:

```sql
INSERT INTO orders (id, user_id, status, created_at) VALUES
    (1, 1, 'completed', '2024-04-01 10:00:00+09'),
    (2, 2, 'completed', '2024-04-03 11:30:00+09'),
    (3, 1, 'pending',   '2024-04-10 09:15:00+09'),
    (4, 4, 'completed', '2024-04-12 14:00:00+09'),
    (5, 6, 'cancelled', '2024-04-15 16:45:00+09');
```

---

# 9. Create Order Items

An order can contain multiple products.

For example:

```text
Order 1
 ├── Product 1 × 1
 └── Product 2 × 2
```

Create the table:

```sql
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL
        REFERENCES orders(id),
    product_id INTEGER NOT NULL
        REFERENCES products(id),
    quantity INTEGER NOT NULL DEFAULT 1,
    unit_price_cents INTEGER NOT NULL
);
```

Insert sample data:

```sql
INSERT INTO order_items
    (order_id, product_id, quantity, unit_price_cents)
VALUES
    (1, 1, 1, 1999),
    (1, 2, 2, 999),
    (2, 4, 1, 8999),
    (3, 5, 1, 2499),
    (4, 1, 1, 1999),
    (4, 3, 1, 4500),
    (5, 2, 1, 999);
```

---

# 10. Create Payments

A payment belongs to an order.

```sql
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL
        REFERENCES orders(id)
        ON DELETE CASCADE,
    amount_cents INTEGER NOT NULL,
    paid_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Insert payments:

```sql
INSERT INTO payments
    (order_id, amount_cents, paid_at)
VALUES
    (1, 3997, '2024-04-01 10:05:00+09'),
    (2, 8999, '2024-04-03 11:35:00+09'),
    (4, 6499, '2024-04-12 14:05:00+09');
```

Notice:

```text
Order 1 → Paid
Order 2 → Paid
Order 3 → Pending → No payment
Order 4 → Paid
Order 5 → Cancelled → No payment
```

This missing-payment situation will become useful when learning `LEFT JOIN`.

---

# 11. Full Setup Script

The complete Phase 2 Session 01 setup can be executed in this order:

```sql
-- 1. Categories
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

INSERT INTO categories (name) VALUES
    ('Books'),
    ('Study Aids'),
    ('Gear');

ALTER TABLE products
ADD COLUMN category_id INTEGER
REFERENCES categories(id);

UPDATE products
SET category_id = 1
WHERE id = 1;

UPDATE products
SET category_id = 2
WHERE id = 2;

UPDATE products
SET category_id = 3
WHERE id IN (3, 4, 5);


-- 2. Orders
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL
        REFERENCES users(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

INSERT INTO orders (id, user_id, status, created_at) VALUES
    (1, 1, 'completed', '2024-04-01 10:00:00+09'),
    (2, 2, 'completed', '2024-04-03 11:30:00+09'),
    (3, 1, 'pending',   '2024-04-10 09:15:00+09'),
    (4, 4, 'completed', '2024-04-12 14:00:00+09'),
    (5, 6, 'cancelled', '2024-04-15 16:45:00+09');


-- 3. Order Items
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL
        REFERENCES orders(id),
    product_id INTEGER NOT NULL
        REFERENCES products(id),
    quantity INTEGER NOT NULL DEFAULT 1,
    unit_price_cents INTEGER NOT NULL
);

INSERT INTO order_items
    (order_id, product_id, quantity, unit_price_cents)
VALUES
    (1, 1, 1, 1999),
    (1, 2, 2, 999),
    (2, 4, 1, 8999),
    (3, 5, 1, 2499),
    (4, 1, 1, 1999),
    (4, 3, 1, 4500),
    (5, 2, 1, 999);


-- 4. Payments
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL
        REFERENCES orders(id)
        ON DELETE CASCADE,
    amount_cents INTEGER NOT NULL,
    paid_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

INSERT INTO payments
    (order_id, amount_cents, paid_at)
VALUES
    (1, 3997, '2024-04-01 10:05:00+09'),
    (2, 8999, '2024-04-03 11:35:00+09'),
    (4, 6499, '2024-04-12 14:05:00+09');
```

---

# 12. Verify the Schema

List all tables:

```text
\dt
```

You should now have:

```text
categories
feedback
order_items
orders
payments
products
users
```

Inspect the relationships:

```text
\d orders
\d order_items
\d payments
\d products
```

For `orders`, you should see a foreign key similar to:

```text
FOREIGN KEY (user_id) REFERENCES users(id)
```

---

# 13. Key Concepts Learned

### Database Design

* Multi-table schemas
* Data duplication
* One-to-many relationships
* Relational database design

### Foreign Keys

* `REFERENCES`
* Referential integrity
* Parent and child tables
* `ON DELETE CASCADE`
* `ON DELETE SET NULL`

### PostgreSQL Commands

* `CREATE TABLE`
* `ALTER TABLE`
* `INSERT`
* `UPDATE`
* `\dt`
* `\d`

### Store Schema

* `users`
* `categories`
* `products`
* `orders`
* `order_items`
* `payments`

---

# 14. Key Takeaway

A real database should not put everything into one giant table.

Instead, related information is separated into focused tables and connected using foreign keys:

```text
users
   ↓
orders
   ↓
order_items
   ↓
products
   ↓
categories

orders
   ↓
payments
```

This reduces duplication, protects data integrity, and creates a structure that can be queried efficiently.

---

