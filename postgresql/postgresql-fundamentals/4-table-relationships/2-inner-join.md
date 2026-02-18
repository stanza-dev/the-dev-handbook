---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-inner-join"
---

# INNER JOIN

INNER JOIN returns only rows that have matching values in both tables.

## Basic Syntax

```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

## Visual Representation

```
    Table A          Table B
  ┌─────────┐      ┌─────────┐
  │    A    │      │    B    │
  │   ┌───┐ │      │ ┌───┐   │
  │   │ ∩ │ │  =   │ │ ∩ │   │  <- INNER JOIN returns this
  │   └───┘ │      │ └───┘   │
  └─────────┘      └─────────┘
```

## Practical Examples

```sql
-- Get books with their author names
SELECT 
    books.title,
    books.published_year,
    authors.name AS author_name
FROM books
INNER JOIN authors ON books.author_id = authors.id;
```

Output:
```
         title          | published_year |  author_name
------------------------+----------------+---------------
 1984                   |           1949 | George Orwell
 Animal Farm            |           1945 | George Orwell
 Clean Code             |           2008 | Robert Martin
```

## Using Table Aliases

```sql
-- Shorter syntax with aliases
SELECT 
    b.title,
    a.name AS author
FROM books b
INNER JOIN authors a ON b.author_id = a.id;
```

## Multiple Joins

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

## JOIN with USING

When column names match in both tables:

```sql
-- Shorter syntax when column names match
SELECT books.title, authors.name
FROM books
INNER JOIN authors USING (author_id);  -- Only if column is named author_id in both

-- Typically used when both have 'id' or same FK name
SELECT *
FROM order_items
INNER JOIN products USING (product_id);
```

## Adding Conditions

```sql
-- Filter joined results
SELECT b.title, a.name
FROM books b
INNER JOIN authors a ON b.author_id = a.id
WHERE b.published_year > 2000
  AND a.country = 'USA'
ORDER BY b.published_year DESC;
```

**Note**: Rows without a match in either table are excluded from results.

📖 [JOIN Types](https://www.postgresql.org/docs/18/queries-table-expressions.html#QUERIES-FROM)

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*