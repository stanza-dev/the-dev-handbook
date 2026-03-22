---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-pg18-features"
---

# PL/pgSQL and PostgreSQL 18 New Features

## Introduction

PostgreSQL 18 introduces several features that enhance PL/pgSQL development. From new built-in functions to changes in trigger behavior, these additions simplify common patterns and improve security. Understanding these changes helps you write more concise, modern PL/pgSQL code.

## Key Concepts

- **OLD/NEW in RETURNING**: DML statements (INSERT, UPDATE, DELETE) can now reference OLD and NEW in their RETURNING clause.
- **AFTER trigger role behavior**: AFTER triggers now run as the role that was active when the triggering event was queued.
- **array_sort()**: A built-in function that sorts array elements in ascending order.
- **array_reverse()**: A built-in function that reverses the order of array elements.
- **uuidv7()**: Generates time-sortable UUIDs (UUID version 7) as a built-in function.

## Real World Context

Before PostgreSQL 18, capturing both old and new values during an UPDATE required a trigger function. Now, `UPDATE ... RETURNING OLD.column, NEW.column` does it in a single statement. Similarly, sorting arrays previously required custom PL/pgSQL functions or unnest/array_agg patterns -- `array_sort()` replaces all of that with a single function call.

## Deep Dive

### OLD/NEW in RETURNING Clauses

Previously, RETURNING could only access the final row state. PostgreSQL 18 adds OLD and NEW references:

```sql
-- See before and after values in one statement
UPDATE products
SET price = price * 1.1
WHERE category = 'electronics'
RETURNING id, OLD.price AS old_price, NEW.price AS new_price;

-- Capture changes in PL/pgSQL
DECLARE
    v_old_price NUMERIC;
    v_new_price NUMERIC;
BEGIN
    UPDATE products SET price = price * 1.1 WHERE id = 42
    RETURNING OLD.price, NEW.price INTO v_old_price, v_new_price;

    RAISE NOTICE 'Price changed from % to %', v_old_price, v_new_price;
END;
```

This reduces the need for triggers that simply capture before/after state.

### AFTER Trigger Role Behavior

In PostgreSQL 18, AFTER triggers run as the role that was active when the triggering events were queued, not the role at commit time:

```sql
-- Previously: if role changed between trigger event and commit,
-- the trigger ran as the role at commit time (confusing)

-- PostgreSQL 18: trigger runs as the role that performed the INSERT/UPDATE/DELETE
-- This is more predictable for security-sensitive triggers
```

This matters in applications that use SET ROLE or SET SESSION AUTHORIZATION within transactions.

### array_sort() and array_reverse()

New built-in functions for array manipulation:

```sql
-- Sort an array
SELECT array_sort(ARRAY[3, 1, 4, 1, 5, 9]);
-- Result: {1, 1, 3, 4, 5, 9}

-- Reverse an array
SELECT array_reverse(ARRAY['c', 'a', 'b']);
-- Result: {b, a, c}

-- Combined: sort descending
SELECT array_reverse(array_sort(ARRAY[3, 1, 4, 1, 5]));
-- Result: {5, 4, 3, 1, 1}

-- In PL/pgSQL
DECLARE
    v_tags TEXT[] := ARRAY['beta', 'alpha', 'gamma'];
BEGIN
    v_tags := array_sort(v_tags);
    -- v_tags is now {'alpha', 'beta', 'gamma'}
END;
```

Before PostgreSQL 18, sorting required: `SELECT array_agg(x ORDER BY x) FROM unnest(arr) x`.

### uuidv7() -- Time-Sortable UUIDs

UUID v7 encodes a timestamp in the most significant bits, making UUIDs naturally sortable by creation time:

```sql
-- Generate a v7 UUID
SELECT uuidv7();
-- Result: 019478a3-b4c5-7def-8901-234567890abc

-- Use as primary key
CREATE TABLE events (
    id UUID PRIMARY KEY DEFAULT uuidv7(),
    name TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- UUIDs are naturally ordered by creation time
-- No need for a separate created_at column for ordering!
SELECT * FROM events ORDER BY id;  -- Chronological order
```

In PL/pgSQL, `uuidv7()` is useful for generating correlation IDs in stored procedures:

```sql
CREATE PROCEDURE batch_process() AS $$
DECLARE
    v_batch_id UUID := uuidv7();
BEGIN
    RAISE NOTICE 'Starting batch %', v_batch_id;
    -- All operations tagged with this batch ID
END;
$$ LANGUAGE plpgsql;
```

## Common Pitfalls

1. **Assuming OLD is available in INSERT RETURNING** -- OLD is only available for UPDATE and DELETE. For INSERT, only NEW exists in RETURNING.
2. **Using uuidv7() where gen_random_uuid() suffices** -- uuidv7() leaks creation time information. Use gen_random_uuid() (v4) when time-ordering is not needed and timing privacy matters.

## Best Practices

1. **Use OLD/NEW in RETURNING to replace simple audit triggers** -- When you only need the before/after values at the call site, RETURNING is simpler than a trigger.
2. **Prefer uuidv7() for primary keys in new tables** -- Time-sortable UUIDs improve B-tree index performance compared to random v4 UUIDs.

## Summary

- PostgreSQL 18 adds OLD/NEW references in RETURNING clauses, reducing the need for change-capture triggers.
- AFTER triggers now run as the role active when events were queued, improving security predictability.
- array_sort(), array_reverse(), and uuidv7() are new built-in functions that replace common workarounds.

## Code Examples

**Demonstrates key PostgreSQL 18 features: OLD/NEW in RETURNING, array_sort(), array_reverse(), and uuidv7().**

```sql
-- PostgreSQL 18: OLD/NEW in RETURNING
UPDATE products SET price = price * 1.1 WHERE id = 42
RETURNING id, OLD.price AS old_price, NEW.price AS new_price;

-- New built-in array functions
SELECT array_sort(ARRAY[3, 1, 4, 1, 5]);    -- {1,1,3,4,5}
SELECT array_reverse(ARRAY['c', 'a', 'b']); -- {b,a,c}

-- Time-sortable UUID v7
SELECT uuidv7();  -- 019478a3-b4c5-7def-...
```


## Resources

- [PostgreSQL 18 Release Notes](https://www.postgresql.org/docs/18/release-18.html) — Full list of PostgreSQL 18 changes and new features

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*