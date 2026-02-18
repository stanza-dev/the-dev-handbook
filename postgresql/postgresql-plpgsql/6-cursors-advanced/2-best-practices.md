---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-best-practices"
---

# Best Practices & Optimization

Write efficient, maintainable PL/pgSQL code.

## Naming Conventions

```plpgsql
-- Prefix parameters to avoid column name conflicts
CREATE FUNCTION get_user(p_user_id INTEGER) -- p_ for parameter
RETURNS users AS $$
DECLARE
    v_user users%ROWTYPE;  -- v_ for variable
BEGIN
    SELECT * INTO v_user 
    FROM users 
    WHERE id = p_user_id;  -- Clear: parameter vs column
    RETURN v_user;
END;
$$ LANGUAGE plpgsql;
```

## Use STRICT for Single-Row Queries

```plpgsql
-- Without STRICT: silently uses first row if multiple
SELECT name INTO user_name FROM users WHERE status = 'active';

-- With STRICT: raises error if not exactly 1 row
SELECT name INTO STRICT user_name FROM users WHERE id = p_id;
```

## Prefer SQL Over Loops

```plpgsql
-- BAD: Row-by-row processing
FOR rec IN SELECT * FROM orders WHERE status = 'pending' LOOP
    UPDATE orders SET status = 'processing' WHERE id = rec.id;
END LOOP;

-- GOOD: Single SQL statement
UPDATE orders SET status = 'processing' WHERE status = 'pending';
```

## Use RETURN QUERY Instead of RETURN NEXT

```plpgsql
-- SLOWER: Building result row by row
FOR rec IN SELECT * FROM users LOOP
    RETURN NEXT rec;
END LOOP;

-- FASTER: Return entire query result
RETURN QUERY SELECT * FROM users;
```

## Minimize Context Switches

```plpgsql
-- BAD: Multiple queries
SELECT name INTO v_name FROM users WHERE id = p_id;
SELECT email INTO v_email FROM users WHERE id = p_id;

-- GOOD: Single query
SELECT name, email INTO v_name, v_email FROM users WHERE id = p_id;
```

## Function Volatility

```plpgsql
-- IMMUTABLE: Same inputs always return same output
-- Can be optimized, cached, used in indexes
CREATE FUNCTION calculate_tax(amount NUMERIC)
RETURNS NUMERIC AS $$
BEGIN
    RETURN amount * 0.08;
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- STABLE: Same result within single query
CREATE FUNCTION get_current_rate()
RETURNS NUMERIC AS $$
BEGIN
    RETURN (SELECT rate FROM config WHERE key = 'tax_rate');
END;
$$ LANGUAGE plpgsql STABLE;

-- VOLATILE: May return different results (default)
CREATE FUNCTION log_access()
RETURNS VOID AS $$
BEGIN
    INSERT INTO access_log (time) VALUES (NOW());
END;
$$ LANGUAGE plpgsql VOLATILE;
```

## Security Best Practices

```plpgsql
-- SECURITY DEFINER: Runs with function owner's privileges
-- Use with caution!
CREATE FUNCTION sensitive_operation()
RETURNS VOID AS $$
BEGIN
    -- Runs as function owner, not caller
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Always set search_path for SECURITY DEFINER
CREATE FUNCTION safe_function()
RETURNS VOID AS $$
BEGIN
    -- operations
END;
$$ LANGUAGE plpgsql 
   SECURITY DEFINER
   SET search_path = public, pg_temp;
```

## Error Handling Guidelines

```plpgsql
-- Be specific with error codes
RAISE EXCEPTION 'User not found'
    USING ERRCODE = 'P0002',  -- no_data_found
          HINT = 'Verify the user ID exists';

-- Log before re-raising
EXCEPTION WHEN OTHERS THEN
    INSERT INTO error_log (func, error) 
    VALUES ('my_function', SQLERRM);
    RAISE;  -- Re-raise original error
```

📖 [Tips for Developing](https://www.postgresql.org/docs/18/plpgsql-development-tips.html)

## Resources

- [Development Tips](https://www.postgresql.org/docs/18/plpgsql-development-tips.html) — Tips for PL/pgSQL development

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*