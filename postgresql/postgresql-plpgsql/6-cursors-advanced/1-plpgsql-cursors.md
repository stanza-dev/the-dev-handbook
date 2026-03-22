---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-cursors"
---

# Working with Cursors

## Introduction

Cursors allow row-by-row processing of query results without loading the entire result set into memory. While set-based SQL is usually preferred, cursors are essential when processing very large datasets, sharing result sets between functions, or navigating forward and backward through results.

## Key Concepts

- **Cursor**: A pointer to a query result set that lets you fetch rows one at a time.
- **Unbound Cursor (REFCURSOR)**: A cursor whose query is defined when it is opened, not in the declaration.
- **Bound Cursor**: A cursor whose query is specified in the DECLARE section.
- **Parameterized Cursor**: A bound cursor that accepts parameters, letting you reuse the same cursor definition with different values.
- **SCROLL Cursor**: A cursor that supports bidirectional navigation (FETCH PRIOR, FETCH FIRST, etc.).

## Real World Context

A data export function needs to stream millions of rows to a client application without consuming gigabytes of server memory. By returning a REFCURSOR, the function lets the client FETCH rows in batches of 1000 within a transaction, processing each batch before fetching the next. This pattern is common in ETL pipelines and report generation.

## Deep Dive

### Declaring Cursors

```sql
DECLARE
    -- Unbound: query defined when opened
    cur REFCURSOR;

    -- Bound: query defined in declaration
    user_cursor CURSOR FOR
        SELECT * FROM users WHERE active = true;

    -- Parameterized: query with parameters
    order_cursor CURSOR (user_id INTEGER) FOR
        SELECT * FROM orders WHERE user_id = $1;
```

### Opening and Fetching

```sql
DECLARE
    rec RECORD;
BEGIN
    OPEN user_cursor;

    LOOP
        FETCH user_cursor INTO rec;
        EXIT WHEN NOT FOUND;
        RAISE NOTICE 'User: %', rec.name;
    END LOOP;

    CLOSE user_cursor;
END;
```

### Fetch Directions (SCROLL cursors)

```sql
FETCH NEXT FROM cur INTO rec;       -- Next row (default)
FETCH PRIOR FROM cur INTO rec;      -- Previous row
FETCH FIRST FROM cur INTO rec;      -- First row
FETCH LAST FROM cur INTO rec;       -- Last row
FETCH ABSOLUTE 5 FROM cur INTO rec; -- 5th row
FETCH RELATIVE -2 FROM cur INTO rec; -- 2 rows back
```

### Returning Cursors (Portal Pattern)

A function can return a cursor that the caller fetches from:

```sql
CREATE FUNCTION get_users_cursor() RETURNS REFCURSOR AS $$
DECLARE
    cur REFCURSOR;
BEGIN
    OPEN cur FOR SELECT * FROM users;
    RETURN cur;
END;
$$ LANGUAGE plpgsql;

-- Usage in a transaction
BEGIN;
SELECT get_users_cursor();
FETCH 10 FROM "<cursor_name>";
COMMIT;  -- Cursor closed
```

### FOR Loop with Cursors

The simplest cursor pattern -- implicit open, fetch, and close:

```sql
DECLARE
    user_cursor CURSOR FOR SELECT * FROM users;
    rec RECORD;
BEGIN
    FOR rec IN user_cursor LOOP
        RAISE NOTICE 'User: %', rec.name;
    END LOOP;
END;
```

## Common Pitfalls

1. **Forgetting to CLOSE cursors** -- Unclosed cursors consume resources until the transaction ends. Always close them in a finally-style pattern or use the FOR loop shorthand which auto-closes.
2. **Using cursors when a simple FOR-IN-SELECT suffices** -- `FOR rec IN SELECT ... LOOP` already processes rows one at a time without the ceremony of DECLARE/OPEN/FETCH/CLOSE.

## Best Practices

1. **Prefer FOR-IN-SELECT over explicit cursors** -- It is shorter, auto-closes, and handles NOT FOUND automatically.
2. **Use parameterized cursors to avoid code duplication** -- If you need the same query with different filter values, parameterized cursors are cleaner than building dynamic SQL.

## Summary

- Cursors process query results row by row without loading everything into memory.
- Bound cursors define the query at declaration; unbound cursors (REFCURSOR) define it at open time.
- The FOR loop shorthand automatically opens, fetches, and closes a cursor, making it the preferred pattern for most use cases.

## Code Examples

**Shows the FOR loop cursor shorthand that automatically handles opening, fetching, and closing the cursor.**

```sql
DECLARE
    user_cursor CURSOR FOR SELECT * FROM users WHERE active = true;
    rec RECORD;
BEGIN
    FOR rec IN user_cursor LOOP
        RAISE NOTICE 'User: % (%)', rec.name, rec.email;
    END LOOP;  -- auto-close
END;
```


## Resources

- [Cursors](https://www.postgresql.org/docs/18/plpgsql-cursors.html) — Cursor documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*