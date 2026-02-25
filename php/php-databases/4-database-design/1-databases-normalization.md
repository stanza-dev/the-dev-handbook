---
source_course: "php-databases"
source_lesson: "php-databases-normalization"
---

# Normalization & Schema Design

## Introduction

Database normalization is the process of organizing tables to reduce data redundancy and prevent update anomalies. Proper schema design is the most impactful decision you make for a database — it determines query performance, data integrity, and how easily the schema evolves.

## Key Concepts

- **Normal Form (NF)**: A set of rules that a table structure must satisfy. Each higher form builds on the previous.
- **Functional Dependency**: Column B depends on column A if each value of A determines exactly one value of B.
- **Denormalization**: Intentionally violating normal forms for performance, trading data integrity for read speed.

## Real World Context

A poorly normalized e-commerce schema might store customer addresses inside the orders table. When the customer moves, you either update every historical order (destroying history) or leave them inconsistent. Proper normalization avoids this dilemma entirely.

## Deep Dive

First Normal Form (1NF) requires atomic values and unique rows:

```sql
-- BAD: Violates 1NF (comma-separated list)
CREATE TABLE orders (
    id INT,
    products VARCHAR(255)  -- 'Widget, Gadget, Thing'
);

-- GOOD: 1NF compliant
CREATE TABLE orders (id INT PRIMARY KEY);
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    PRIMARY KEY (order_id, product_id)
);
```

Second Normal Form (2NF) eliminates partial dependencies on composite keys:

```sql
-- BAD: product_name depends only on product_id, not the full key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(255),  -- Partial dependency!
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- GOOD: Separate products table
CREATE TABLE products (id INT PRIMARY KEY, name VARCHAR(255));
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

Third Normal Form (3NF) eliminates transitive dependencies:

```sql
-- BAD: city depends on zip_code, not directly on user
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    zip_code VARCHAR(10),
    city VARCHAR(255)  -- Transitive: user -> zip -> city
);

-- GOOD: Separate locations
CREATE TABLE locations (
    zip_code VARCHAR(10) PRIMARY KEY,
    city VARCHAR(255)
);
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    zip_code VARCHAR(10) REFERENCES locations(zip_code)
);
```

Sometimes denormalization is the right trade-off. Caching computed values avoids expensive joins:

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    total_amount DECIMAL(10,2),  -- Cached sum of order_items
    item_count INT               -- Cached COUNT of order_items
);
```

This is acceptable when you update the cached values within the same transaction as the source data changes.

Proper indexing complements normalization:

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE FULLTEXT INDEX idx_products_search ON products(name, description);
```

Composite indexes are read left-to-right. An index on `(user_id, status)` helps queries filtering by `user_id` alone or `user_id AND status`, but not `status` alone.

## Common Pitfalls

1. **Storing comma-separated values in a single column** — This violates 1NF, makes querying individual values impossible with indexes, and breaks referential integrity.
2. **Over-normalizing** — Normalizing to 5NF when 3NF suffices adds unnecessary joins without meaningful data integrity benefit.

## Best Practices

1. **Start with 3NF and denormalize only when measured** — Profile your queries first. Denormalize only after EXPLAIN shows that joins are the actual bottleneck.
2. **Design composite indexes to match your WHERE clause order** — The leftmost columns of a composite index must match your query filters for the index to be used.

## Summary

- 1NF requires atomic values; 2NF eliminates partial dependencies; 3NF eliminates transitive dependencies.
- Denormalization is a deliberate trade-off, not a shortcut — cache computed values in the same transaction.
- Composite indexes are read left-to-right and must match your WHERE clause column order.
- Start at 3NF and denormalize only when profiling reveals a need.

## Code Examples

**Normalized schema with proper indexes**

```php
<?php
declare(strict_types=1);

// Schema creation with proper normalization and indexing
$pdo->exec('
    CREATE TABLE categories (
        id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
        parent_id INT UNSIGNED NULL,
        name VARCHAR(255) NOT NULL,
        slug VARCHAR(255) NOT NULL UNIQUE,
        FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL
    )
');

$pdo->exec('
    CREATE TABLE products (
        id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
        category_id INT UNSIGNED,
        sku VARCHAR(50) NOT NULL UNIQUE,
        name VARCHAR(255) NOT NULL,
        price DECIMAL(10, 2) NOT NULL,
        stock INT UNSIGNED DEFAULT 0,
        FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL,
        INDEX idx_category (category_id),
        FULLTEXT INDEX idx_search (name)
    )
');
?>
```


## Resources

- [Database Normalization](https://en.wikipedia.org/wiki/Database_normalization) — Comprehensive overview of database normalization forms

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*