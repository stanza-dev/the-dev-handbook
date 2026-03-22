---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-loops"
---

# Loop Structures

## Introduction

PL/pgSQL provides several loop types for iteration, from simple counting loops to query-driven iteration over result sets. While you should prefer set-based SQL when possible, loops are essential for row-by-row processing that requires procedural logic -- such as sending notifications per user or applying different business rules to each row.

## Key Concepts

- **Basic LOOP**: An unconditional infinite loop that must be exited explicitly with `EXIT` or `EXIT WHEN`.
- **WHILE Loop**: Repeats as long as a condition is true, checked before each iteration.
- **FOR Loop (Integer)**: Iterates over a numeric range with optional REVERSE and BY (step) clauses.
- **FOR Loop (Query)**: Iterates over the result rows of a SQL query.
- **FOREACH**: Iterates over elements of an array.
- **Labeled Loop**: A loop prefixed with `<<label>>` so that EXIT and CONTINUE can target a specific nesting level.

## Real World Context

A data migration script may need to process millions of rows in batches of 1000 to avoid locking the entire table. A FOR loop over a query with LIMIT/OFFSET, combined with intermediate COMMITs inside a procedure, handles this cleanly. Labeled loops let you break out of nested iterations -- for example, stopping an outer batch loop when a fatal error is detected in an inner row loop.

## Deep Dive

### Basic LOOP

```sql
CREATE FUNCTION countdown(start_num INTEGER) RETURNS TEXT AS $$
DECLARE
    counter INTEGER := start_num;
    result TEXT := '';
BEGIN
    LOOP
        result := result || counter::TEXT || ' ';
        counter := counter - 1;
        EXIT WHEN counter < 0;
    END LOOP;
    RETURN result;
END;
$$ LANGUAGE plpgsql;
```

### WHILE Loop

```sql
CREATE FUNCTION factorial(n INTEGER) RETURNS BIGINT AS $$
DECLARE
    result BIGINT := 1;
    counter INTEGER := n;
BEGIN
    WHILE counter > 1 LOOP
        result := result * counter;
        counter := counter - 1;
    END LOOP;
    RETURN result;
END;
$$ LANGUAGE plpgsql;
```

### FOR Loop (Integer Range)

```sql
FOR i IN 1..10 LOOP
    -- ascending: 1, 2, 3, ..., 10
END LOOP;

FOR i IN REVERSE 10..1 LOOP
    -- descending: 10, 9, ..., 1
END LOOP;

FOR i IN 1..10 BY 2 LOOP
    -- stepping: 1, 3, 5, 7, 9
END LOOP;
```

### FOR Loop (Query Results)

```sql
CREATE FUNCTION process_users() RETURNS VOID AS $$
DECLARE
    user_rec RECORD;
BEGIN
    FOR user_rec IN SELECT id, name, email FROM users WHERE active = true LOOP
        RAISE NOTICE 'Processing: % (%)', user_rec.name, user_rec.email;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

### FOREACH (Arrays)

```sql
CREATE FUNCTION sum_array(arr INTEGER[]) RETURNS INTEGER AS $$
DECLARE
    element INTEGER;
    total INTEGER := 0;
BEGIN
    FOREACH element IN ARRAY arr LOOP
        total := total + element;
    END LOOP;
    RETURN total;
END;
$$ LANGUAGE plpgsql;
```

### Loop Control and Labels

```sql
<<outer_loop>>
FOR i IN 1..10 LOOP
    <<inner_loop>>
    FOR j IN 1..10 LOOP
        IF some_condition THEN
            EXIT outer_loop;  -- Exit the outer loop entirely
        END IF;
        CONTINUE WHEN j = 5;  -- Skip rest of inner iteration
    END LOOP inner_loop;
END LOOP outer_loop;
```

## Common Pitfalls

1. **Forgetting EXIT in a basic LOOP** -- Without EXIT or EXIT WHEN, a basic LOOP runs forever and your function never returns.
2. **Using loops where a single SQL statement suffices** -- `UPDATE orders SET status = 'done' WHERE status = 'pending'` is far faster than looping row by row.

## Best Practices

1. **Prefer set-based SQL over loops** -- Only use loops when each iteration requires different procedural logic that cannot be expressed in SQL.
2. **Use labeled loops for nested iteration** -- Labels make it clear which loop EXIT and CONTINUE target, improving readability.

## Summary

- PL/pgSQL offers basic LOOP, WHILE, FOR (integer and query), and FOREACH (array) loop types.
- EXIT and CONTINUE control loop flow; labels let you target specific nesting levels.
- Always prefer a single SQL statement over a loop when the operation can be expressed in set-based logic.

## Code Examples

**Uses FOREACH to iterate over array elements, demonstrating a common loop pattern in PL/pgSQL.**

```sql
CREATE FUNCTION sum_array(arr INTEGER[]) RETURNS INTEGER AS $$
DECLARE
    element INTEGER;
    total INTEGER := 0;
BEGIN
    FOREACH element IN ARRAY arr LOOP
        total := total + element;
    END LOOP;
    RETURN total;
END;
$$ LANGUAGE plpgsql;

SELECT sum_array(ARRAY[1, 2, 3, 4, 5]);  -- Returns 15
```


## Resources

- [Loop Statements](https://www.postgresql.org/docs/18/plpgsql-control-structures.html#PLPGSQL-CONTROL-STRUCTURES-LOOPS) — Complete loop documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*