---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-select-into"
---

# Executing Queries in PL/pgSQL

## Introduction

PL/pgSQL seamlessly integrates with SQL queries, letting you execute SELECT, INSERT, UPDATE, and DELETE statements and capture their results into variables. This tight integration is what makes PL/pgSQL powerful -- you combine procedural control flow with the full power of SQL without leaving the database engine.

## Key Concepts

- **SELECT INTO**: Captures the result of a query into one or more PL/pgSQL variables.
- **STRICT**: An optional modifier that raises an error if SELECT INTO returns zero rows or more than one row.
- **FOUND**: A boolean variable automatically set after most SQL statements -- TRUE if a row was returned/affected, FALSE otherwise.
- **PERFORM**: Executes a query and discards the result. Used when you call a function for its side effects.
- **GET DIAGNOSTICS**: Retrieves metadata about the last executed command, such as the number of affected rows.

## Real World Context

A user registration function needs to: (1) check if the email already exists, (2) insert the new user, and (3) return the generated ID. SELECT INTO with FOUND handles the existence check, INSERT ... RETURNING INTO captures the new ID, and GET DIAGNOSTICS can verify exactly how many rows were affected -- all in one clean function.

## Deep Dive

### SELECT INTO

```sql
DECLARE
    user_name TEXT;
    user_email TEXT;
    user_count INTEGER;
BEGIN
    SELECT COUNT(*) INTO user_count FROM users;

    SELECT name, email INTO user_name, user_email
    FROM users WHERE id = 1;

    -- STRICT: raises error if not exactly 1 row
    SELECT name INTO STRICT user_name
    FROM users WHERE id = 1;
END;
```

### Checking for Found Rows

```sql
CREATE FUNCTION get_user_name(user_id INTEGER) RETURNS TEXT AS $$
DECLARE
    result TEXT;
BEGIN
    SELECT name INTO result FROM users WHERE id = user_id;

    IF NOT FOUND THEN
        RETURN 'User not found';
    END IF;

    RETURN result;
END;
$$ LANGUAGE plpgsql;
```

### PERFORM (No Results Needed)

Use PERFORM when you want to execute a query but do not need its result:

```sql
PERFORM pg_notify('channel', 'message');

PERFORM 1 FROM users WHERE email = input_email;
IF FOUND THEN
    RAISE EXCEPTION 'Email already exists';
END IF;
```

### INSERT with RETURNING INTO

```sql
CREATE FUNCTION create_order(
    p_user_id INTEGER,
    p_items JSONB
) RETURNS INTEGER AS $$
DECLARE
    new_order_id INTEGER;
BEGIN
    INSERT INTO orders (user_id, status, created_at)
    VALUES (p_user_id, 'pending', NOW())
    RETURNING id INTO new_order_id;

    INSERT INTO order_items (order_id, product_id, quantity)
    SELECT new_order_id, (item->>'product_id')::INTEGER, (item->>'quantity')::INTEGER
    FROM jsonb_array_elements(p_items) AS item;

    RETURN new_order_id;
END;
$$ LANGUAGE plpgsql;
```

### GET DIAGNOSTICS

```sql
DECLARE
    row_count INTEGER;
BEGIN
    UPDATE products SET price = price * 1.1 WHERE category = 'electronics';
    GET DIAGNOSTICS row_count = ROW_COUNT;
    RAISE NOTICE 'Updated % rows', row_count;
END;
```

## Common Pitfalls

1. **Forgetting to check FOUND** -- SELECT INTO silently sets variables to NULL when no row matches. Always check FOUND after SELECT INTO to avoid operating on NULL values.
2. **Using SELECT without INTO inside PL/pgSQL** -- A bare SELECT (without INTO) causes an error. Use PERFORM if you do not need the result.

## Best Practices

1. **Use STRICT for single-row lookups** -- It makes the intent clear and catches bugs where a query unexpectedly returns zero or multiple rows.
2. **Use RETURNING INTO instead of a separate SELECT** -- Combining INSERT/UPDATE with RETURNING INTO is atomic and avoids an extra query.

## Summary

- SELECT INTO captures query results into variables; add STRICT to enforce exactly one row.
- The FOUND variable indicates whether the last SQL statement returned or affected any rows.
- PERFORM discards results and is used for side-effect-only calls; GET DIAGNOSTICS retrieves row counts.

## Code Examples

**Demonstrates SELECT INTO with the FOUND variable to detect and handle missing rows gracefully.**

```sql
CREATE FUNCTION get_user_name(user_id INTEGER) RETURNS TEXT AS $$
DECLARE
    result TEXT;
BEGIN
    SELECT name INTO result FROM users WHERE id = user_id;

    IF NOT FOUND THEN
        RETURN 'User not found';
    END IF;

    RETURN result;
END;
$$ LANGUAGE plpgsql;
```


## Resources

- [PL/pgSQL Statements](https://www.postgresql.org/docs/18/plpgsql-statements.html) — Basic statements in PL/pgSQL

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*