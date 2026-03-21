---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-outer-joins"
---

# OUTER JOINs

## Introduction
While INNER JOIN only returns matching rows, OUTER JOINs include rows even when there is no match in the other table. This is essential for answering questions like "which customers have never ordered?" or "show all categories, even those with no products."

## Key Concepts
- **LEFT JOIN**: Returns all rows from the left table, with matching rows from the right table (or NULL if no match exists).
- **RIGHT JOIN**: Returns all rows from the right table, with matching rows from the left table (or NULL if no match exists).
- **FULL OUTER JOIN**: Returns all rows from both tables, matching where possible and filling NULLs elsewhere.
- **IS NULL filter pattern**: After a LEFT JOIN, `WHERE right_table.id IS NULL` finds rows in the left table that have no match in the right table.

## Real World Context
LEFT JOIN is the second most common JOIN type after INNER JOIN. Use it whenever you need "all X, even those without Y": all products even those never ordered, all users even those who never logged in, all categories even empty ones. The IS NULL pattern is the standard way to find "missing" relationships.

## Deep Dive

### LEFT JOIN (LEFT OUTER JOIN)

LEFT JOIN returns ALL rows from the left table, with matching rows from the right (or NULL):

```
    Table A          Table B
  ┌─────────┐      ┌─────────┐
  │XXXXXXXXX│      │    B    │
  │XXX┌───┐X│      │ ┌───┐   │
  │XXX│ X │X│      │ │ X │   │  <- LEFT JOIN returns shaded area
  │XXX└───┘X│      │ └───┘   │
  └─────────┘      └─────────┘
```

Here is a practical example:

```sql
-- All authors, even those without books
SELECT 
    a.name AS author,
    b.title AS book
FROM authors a
LEFT JOIN books b ON a.id = b.author_id;
```

Authors without books appear in the result with NULL in the book column:

```
     author      |      book
-----------------+----------------
 George Orwell   | 1984
 George Orwell   | Animal Farm
 J.D. Salinger   | NULL           <- No books for this author
```

This would not be possible with INNER JOIN, which would exclude J.D. Salinger entirely.

### RIGHT JOIN (RIGHT OUTER JOIN)

RIGHT JOIN returns ALL rows from the right table:

```sql
-- All books, even those without authors (rare)
SELECT 
    a.name AS author,
    b.title AS book
FROM authors a
RIGHT JOIN books b ON a.id = b.author_id;
```

RIGHT JOIN is rarely used in practice because you can always rewrite it as a LEFT JOIN by swapping the table order.

### FULL OUTER JOIN

FULL OUTER JOIN returns ALL rows from BOTH tables:

```sql
-- All authors and all books, matching where possible
SELECT 
    a.name AS author,
    b.title AS book
FROM authors a
FULL OUTER JOIN books b ON a.id = b.author_id;
```

Rows from both sides that have no match are included with NULLs. This is useful for finding mismatches between two datasets.

### Finding Missing Relationships

The LEFT JOIN + IS NULL pattern finds rows without matches:

```sql
-- Find authors without any books
SELECT a.name
FROM authors a
LEFT JOIN books b ON a.id = b.author_id
WHERE b.id IS NULL;

-- Find orphaned books (no author)
SELECT b.title
FROM books b
LEFT JOIN authors a ON b.author_id = a.id
WHERE a.id IS NULL;
```

This pattern is used constantly — finding inactive users, unsold products, unassigned tickets, etc.

### Practical Example: Report with All Categories

LEFT JOIN ensures all categories appear, even those with no sales:

```sql
-- Sales report showing all categories, even with no sales
SELECT 
    c.name AS category,
    COALESCE(SUM(oi.quantity * p.price), 0) AS total_sales
FROM categories c
LEFT JOIN products p ON c.id = p.category_id
LEFT JOIN order_items oi ON p.id = oi.product_id
GROUP BY c.name
ORDER BY total_sales DESC;
```

COALESCE replaces NULL sums with 0 for categories that have no sales.

## Common Pitfalls
1. **Using INNER JOIN when you need LEFT JOIN** — If you want all rows from one table regardless of matches, use LEFT JOIN. INNER JOIN silently drops non-matching rows.
2. **WHERE filtering defeats LEFT JOIN** — Adding `WHERE b.published_year > 2000` after a LEFT JOIN converts it into an INNER JOIN because NULL values do not satisfy the condition. Put such filters in the ON clause instead.

## Best Practices
1. **Prefer LEFT JOIN over RIGHT JOIN** — RIGHT JOIN can always be rewritten as LEFT JOIN by swapping tables. LEFT JOIN is more common and easier to read.
2. **Use COALESCE with LEFT JOIN aggregates** — SUM, COUNT, and other aggregates return NULL for groups with no matches. Use COALESCE to replace NULL with 0.

## Summary
- LEFT JOIN returns all rows from the left table, with NULLs for non-matching right table columns.
- RIGHT JOIN is the reverse but rarely used — rewrite it as LEFT JOIN instead.
- FULL OUTER JOIN returns all rows from both tables.
- Use LEFT JOIN + WHERE IS NULL to find rows without matches (e.g., customers without orders).
- Be careful with WHERE clauses on LEFT JOIN results — they can accidentally convert the join to an INNER JOIN.

## Code Examples

**LEFT JOIN + IS NULL pattern to find users without orders — the most common way to find missing relationships**

```sql
-- Find users who have never placed an order
SELECT u.id, u.name, u.email
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL
ORDER BY u.created_at DESC;
```


## Resources

- [JOIN Types](https://www.postgresql.org/docs/18/queries-table-expressions.html#QUERIES-JOIN) — Official documentation on all PostgreSQL JOIN types

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*