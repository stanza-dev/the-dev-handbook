---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-select-into"
---

# Executing Queries in PL/pgSQL

PL/pgSQL seamlessly integrates with SQL queries.

## SELECT INTO

Capture query results into variables:

```plpgsql
DECLARE
    user_name TEXT;
    user_email TEXT;
    user_count INTEGER;
BEGIN
    -- Single row, single value
    SELECT COUNT(*) INTO user_count FROM users;
    
    -- Single row, multiple values
    SELECT name, email INTO user_name, user_email 
    FROM users WHERE id = 1;
    
    -- Using STRICT (raises error if not exactly 1 row)
    SELECT name INTO STRICT user_name 
    FROM users WHERE id = 1;
END;
```

## Checking for Found Rows

```plpgsql
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

The `FOUND` variable is set by:
- SELECT INTO: true if row(s) returned
- INSERT/UPDATE/DELETE: true if row(s) affected
- FOR loops: true if at least one iteration executed

## INTO with Row Type

```plpgsql
CREATE FUNCTION get_user(user_id INTEGER) RETURNS users AS $$
DECLARE
    result users%ROWTYPE;
BEGIN
    SELECT * INTO result FROM users WHERE id = user_id;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'User % not found', user_id;
    END IF;
    
    RETURN result;
END;
$$ LANGUAGE plpgsql;
```

## PERFORM (No Results Needed)

Execute query without capturing results:

```plpgsql
-- Just execute, ignore result
PERFORM update_timestamp() WHERE id = user_id;

-- Useful for calling functions for side effects
PERFORM pg_notify('channel', 'message');

-- Or when you just need to check EXISTS
PERFORM 1 FROM users WHERE email = input_email;
IF FOUND THEN
    RAISE EXCEPTION 'Email already exists';
END IF;
```

## Modifying Data

```plpgsql
CREATE FUNCTION create_order(
    p_user_id INTEGER,
    p_items JSONB
) RETURNS INTEGER AS $$
DECLARE
    new_order_id INTEGER;
BEGIN
    -- INSERT with RETURNING
    INSERT INTO orders (user_id, status, created_at)
    VALUES (p_user_id, 'pending', NOW())
    RETURNING id INTO new_order_id;
    
    -- INSERT multiple rows (from JSON)
    INSERT INTO order_items (order_id, product_id, quantity)
    SELECT new_order_id, (item->>'product_id')::INTEGER, (item->>'quantity')::INTEGER
    FROM jsonb_array_elements(p_items) AS item;
    
    RETURN new_order_id;
END;
$$ LANGUAGE plpgsql;
```

## GET DIAGNOSTICS

Get information about last command:

```plpgsql
DECLARE
    row_count INTEGER;
BEGIN
    UPDATE products SET price = price * 1.1 WHERE category = 'electronics';
    
    GET DIAGNOSTICS row_count = ROW_COUNT;
    
    RAISE NOTICE 'Updated % rows', row_count;
END;
```

📖 [Basic Statements](https://www.postgresql.org/docs/18/plpgsql-statements.html)

## Resources

- [PL/pgSQL Statements](https://www.postgresql.org/docs/18/plpgsql-statements.html) — Basic statements in PL/pgSQL

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*