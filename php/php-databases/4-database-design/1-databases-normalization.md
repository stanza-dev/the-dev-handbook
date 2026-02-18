---
source_course: "php-databases"
source_lesson: "php-databases-normalization"
---

# Normalization & Schema Design

Proper database design prevents data anomalies and improves performance.

## Normal Forms

### First Normal Form (1NF)
- Atomic values (no arrays/lists in columns)
- Each row is unique

```sql
-- BAD: Violates 1NF
CREATE TABLE orders (
    id INT,
    products VARCHAR(255)  -- 'Widget, Gadget, Thing'
);

-- GOOD: 1NF compliant
CREATE TABLE orders (id INT PRIMARY KEY);
CREATE TABLE order_items (
    order_id INT,
    product_id INT
);
```

### Second Normal Form (2NF)
- In 1NF
- No partial dependencies (all non-key columns depend on entire primary key)

```sql
-- BAD: product_name depends only on product_id
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(255),  -- Partial dependency!
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- GOOD: Separate products table
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

### Third Normal Form (3NF)
- In 2NF
- No transitive dependencies

```sql
-- BAD: city depends on zip_code, not directly on user
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    zip_code VARCHAR(10),
    city VARCHAR(255)  -- Transitive dependency!
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

## Denormalization for Performance

```sql
-- Sometimes denormalization is acceptable
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    user_name VARCHAR(255),  -- Denormalized for display
    total_amount DECIMAL(10,2),  -- Cached calculation
    item_count INT  -- Cached count
);
```

## Indexing Strategies

```sql
-- Primary key (automatic index)
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT
);

-- Foreign key index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Composite index for common queries
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Unique constraint with index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Full-text search
CREATE FULLTEXT INDEX idx_products_search ON products(name, description);
```

## Resources

- [MySQL CREATE INDEX](https://dev.mysql.com/doc/refman/8.0/en/create-index.html) — MySQL indexing documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*