---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-inner-join"
---

# INNER JOIN

## Introduction
INNER JOIN combines rows from two or more tables based on a related column. It returns only the rows that have matching values in both tables. This is the most commonly used JOIN type and the foundation of querying relational data.

## Key Concepts
- **INNER JOIN**: Returns only rows where a match exists in both tables. Rows without a match in either table are excluded.
- **ON clause**: Specifies the join condition — typically matching a foreign key to a primary key.
- **Table aliases**: Short names (like `b` for `books`) that make multi-table queries more readable.
- **USING clause**: A shorthand for ON when both tables have a column with the same name.

## Real World Context
Almost every database query in a real application involves at least one JOIN. Displaying an order with the customer's name, showing a blog post with the author's info, or listing products with their categories — all require INNER JOINs. Knowing JOIN syntax is as fundamental as knowing SELECT.

## Deep Dive

### Basic Syntax

INNER JOIN connects two tables on a matching condition:

```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

The ON clause defines how the tables relate to each other.

### Visual Representation

INNER JOIN returns only the intersection — rows that exist in both tables:

```
    Table A          Table B
  ┌─────────┐      ┌─────────┐
  │    A    │      │    B    │
  │   ┌───┐ │      │ ┌───┐   │
  │   │ X │ │  =   │ │ X │   │  <- INNER JOIN returns X (intersection)
  │   └───┘ │      │ └───┘   │
  └─────────┘      └─────────┘
```

Rows in A without a match in B are excluded, and vice versa.

### Practical Example

Join books with their authors to display both in one result:

```sql
-- Get books with their author names
SELECT 
    books.title,
    books.published_year,
    authors.name AS author_name
FROM books
INNER JOIN authors ON books.author_id = authors.id;
```

This returns one row per book, with the author's name included. Books without a valid author_id are excluded.

### Using Table Aliases

Aliases make queries shorter and more readable:

```sql
-- Shorter syntax with aliases
SELECT 
    b.title,
    a.name AS author
FROM books b
INNER JOIN authors a ON b.author_id = a.id;
```

The alias `b` replaces `books` and `a` replaces `authors` throughout the query.

### Multiple Joins

You can chain multiple JOINs to connect three or more tables:

```sql
-- Get order details with user and product info
SELECT 
    o.id AS order_id,
    u.name AS customer,
    p.name AS product,
    oi.quantity,
    oi.quantity * p.price AS line_total
FROM orders o
INNER JOIN users u ON o.user_id = u.id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;
```

Each JOIN adds another table to the result, connecting through foreign key relationships.

### JOIN with USING

When both tables have a column with the same name, USING simplifies the syntax:

```sql
-- USING shorthand when column names match
SELECT *
FROM order_items
INNER JOIN products USING (product_id);
```

USING (product_id) is equivalent to ON order_items.product_id = products.product_id, but shorter.

### Adding WHERE Conditions

Filter the joined results with WHERE:

```sql
-- Filter joined results
SELECT b.title, a.name
FROM books b
INNER JOIN authors a ON b.author_id = a.id
WHERE b.published_year > 2000
  AND a.country = 'USA'
ORDER BY b.published_year DESC;
```

WHERE runs after the JOIN, so it filters the combined result set.

## Common Pitfalls
1. **Forgetting the ON clause** — Omitting ON creates a CROSS JOIN (every combination of rows), which can produce millions of rows and crash your query.
2. **Ambiguous column names** — If both tables have a column with the same name (like `id`), you must qualify it with the table name: `books.id`, not just `id`.

## Best Practices
1. **Always use table aliases in JOINs** — Aliases make queries more readable and prevent ambiguity.
2. **Be explicit about JOIN type** — Write `INNER JOIN` instead of just `JOIN`. Even though they are equivalent, being explicit improves readability.
3. **Join on indexed columns** — Foreign keys and primary keys should be indexed for fast JOIN performance.

## Summary
- INNER JOIN returns only rows with matching values in both tables.
- Use ON to specify the join condition (typically foreign key = primary key).
- Table aliases (e.g., `b` for `books`) improve readability.
- USING is a shorthand when both tables share a column name.
- Chain multiple JOINs to connect three or more tables.
- Rows without a match in either table are excluded from the result.

## Code Examples

**Joining four tables to build an order details report — each JOIN connects through a foreign key relationship**

```sql
-- Multi-table INNER JOIN with aliases
SELECT 
    o.id AS order_id,
    u.name AS customer,
    p.name AS product,
    oi.quantity,
    oi.quantity * p.price AS line_total
FROM orders o
INNER JOIN users u ON o.user_id = u.id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id
WHERE o.created_at >= '2024-01-01';
```


## Resources

- [JOIN Types](https://www.postgresql.org/docs/18/queries-table-expressions.html#QUERIES-FROM) — Official documentation on JOIN syntax and behavior

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*