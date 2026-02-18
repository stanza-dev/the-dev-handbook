---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-outer-joins"
---

# OUTER JOINs

OUTER JOINs include rows even when there's no match in the other table.

## LEFT JOIN (LEFT OUTER JOIN)

Returns ALL rows from the left table, with matching rows from the right (or NULL):

```
    Table A          Table B
  ┌─────────┐      ┌─────────┐
  │█████████│      │    B    │
  │███┌───┐█│      │ ┌───┐   │
  │███│ ∩ │█│      │ │ ∩ │   │  <- LEFT JOIN returns shaded area
  │███└───┘█│      │ └───┘   │
  └─────────┘      └─────────┘
```

```sql
-- All authors, even those without books
SELECT 
    a.name AS author,
    b.title AS book
FROM authors a
LEFT JOIN books b ON a.id = b.author_id;
```

Output:
```
     author      |      book
-----------------+----------------
 George Orwell   | 1984
 George Orwell   | Animal Farm
 J.D. Salinger   | NULL           <- No books for this author
```

## RIGHT JOIN (RIGHT OUTER JOIN)

Returns ALL rows from the right table:

```sql
-- All books, even those without authors (rare)
SELECT 
    a.name AS author,
    b.title AS book
FROM authors a
RIGHT JOIN books b ON a.id = b.author_id;
```

**Tip**: RIGHT JOIN is rarely used. You can always rewrite it as LEFT JOIN by swapping the tables.

## FULL OUTER JOIN

Returns ALL rows from BOTH tables:

```
    Table A          Table B
  ┌─────────┐      ┌─────────┐
  │█████████│      │█████████│
  │███┌───┐█│      │█┌───┐███│
  │███│ ∩ │█│      │█│ ∩ │███│  <- FULL JOIN returns all shaded
  │███└───┘█│      │█└───┘███│
  └─────────┘      └─────────┘
```

```sql
-- All authors and all books, matching where possible
SELECT 
    a.name AS author,
    b.title AS book
FROM authors a
FULL OUTER JOIN books b ON a.id = b.author_id;
```

## Finding Missing Relationships

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

## Practical Example: Report with All Categories

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

📖 [Join Types](https://www.postgresql.org/docs/18/queries-table-expressions.html#QUERIES-JOIN)

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*