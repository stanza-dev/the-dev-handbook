---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-returning-data"
---

# Returning Data from Functions

## Introduction

Functions can return scalar values, composite rows, or entire result sets. Choosing the right return mechanism determines how callers consume your function -- as a value in an expression, as a virtual table in FROM clauses, or as multiple named outputs. Understanding these options is key to designing clean, composable database APIs.

## Key Concepts

- **Scalar RETURNS**: The function returns a single value of a given type.
- **RETURNS SETOF**: The function returns zero or more rows, making it callable in FROM clauses like a table.
- **RETURN NEXT**: Adds one row to the result set of a SETOF function and continues execution.
- **RETURN QUERY**: Appends an entire query result to the SETOF output in one operation.
- **RETURNS TABLE(...)**: Defines custom output columns for a set-returning function.
- **OUT Parameters**: An alternative way to return multiple named values without defining a custom type.

## Real World Context

A reporting dashboard calls a function that returns a table of sales metrics: total revenue, order count, and average order value, broken down by region. Using RETURNS TABLE with clearly named columns makes the function behave like a view that accepts parameters -- the frontend team can query it with standard SQL and apply further filtering.

## Deep Dive

### Scalar Returns

```sql
CREATE FUNCTION get_total_orders(user_id INTEGER) RETURNS INTEGER AS $$
DECLARE
    total INTEGER;
BEGIN
    SELECT COUNT(*) INTO total FROM orders WHERE user_id = $1;
    RETURN total;
END;
$$ LANGUAGE plpgsql;
```

### SETOF: Returning Multiple Rows

Using RETURN NEXT (row by row):

```sql
CREATE FUNCTION get_active_users() RETURNS SETOF users AS $$
DECLARE
    user_rec users%ROWTYPE;
BEGIN
    FOR user_rec IN SELECT * FROM users WHERE active = true LOOP
        RETURN NEXT user_rec;
    END LOOP;
    RETURN;
END;
$$ LANGUAGE plpgsql;
```

Using RETURN QUERY (set-based, preferred):

```sql
CREATE FUNCTION search_products(search_term TEXT)
RETURNS SETOF products AS $$
BEGIN
    RETURN QUERY
        SELECT * FROM products
        WHERE name ILIKE '%' || search_term || '%';
    RETURN;
END;
$$ LANGUAGE plpgsql;
```

### TABLE Return Type

Define custom output columns:

```sql
CREATE FUNCTION get_order_summary(p_user_id INTEGER)
RETURNS TABLE(
    order_id INTEGER,
    total NUMERIC,
    item_count BIGINT,
    order_date DATE
) AS $$
BEGIN
    RETURN QUERY
    SELECT o.id, o.total, COUNT(oi.id), o.created_at::DATE
    FROM orders o
    LEFT JOIN order_items oi ON o.id = oi.order_id
    WHERE o.user_id = p_user_id
    GROUP BY o.id, o.total, o.created_at;
END;
$$ LANGUAGE plpgsql;
```

### OUT Parameters

```sql
CREATE FUNCTION get_stats(
    IN p_user_id INTEGER,
    OUT order_count INTEGER,
    OUT total_spent NUMERIC,
    OUT avg_order NUMERIC
) AS $$
BEGIN
    SELECT COUNT(*), SUM(total), AVG(total)
    INTO order_count, total_spent, avg_order
    FROM orders WHERE user_id = p_user_id;
END;
$$ LANGUAGE plpgsql;

SELECT * FROM get_stats(42);
```

## Common Pitfalls

1. **Using RETURN NEXT in a loop when RETURN QUERY would suffice** -- RETURN NEXT processes rows one at a time and is slower. Use RETURN QUERY for set-based results.
2. **Forgetting the final RETURN in SETOF functions** -- Without a bare RETURN at the end, some PL/pgSQL versions may produce warnings.

## Best Practices

1. **Use RETURNS TABLE for custom result shapes** -- It is self-documenting and avoids the need for custom types.
2. **Prefer RETURN QUERY over RETURN NEXT** -- It processes all rows in one operation with less overhead.

## Summary

- Scalar RETURNS is for single values; SETOF and TABLE are for multi-row results that callers use like virtual tables.
- RETURN QUERY is faster and cleaner than RETURN NEXT for set-based operations.
- OUT parameters offer a lightweight way to return multiple named values without creating a custom type.

## Code Examples

**A TABLE-returning function that provides a parameterized view of order summaries with custom output columns.**

```sql
CREATE FUNCTION get_order_summary(p_user_id INTEGER)
RETURNS TABLE(
    order_id INTEGER,
    total NUMERIC,
    item_count BIGINT,
    order_date DATE
) AS $$
BEGIN
    RETURN QUERY
    SELECT o.id, o.total, COUNT(oi.id), o.created_at::DATE
    FROM orders o
    LEFT JOIN order_items oi ON o.id = oi.order_id
    WHERE o.user_id = p_user_id
    GROUP BY o.id, o.total, o.created_at;
END;
$$ LANGUAGE plpgsql;
```


## Resources

- [Returning From a Function](https://www.postgresql.org/docs/18/plpgsql-control-structures.html#PLPGSQL-STATEMENTS-RETURNING) — RETURN, RETURN NEXT, RETURN QUERY documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*