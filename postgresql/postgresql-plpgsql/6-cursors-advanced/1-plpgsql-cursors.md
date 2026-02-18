---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-cursors"
---

# Working with Cursors

Cursors allow row-by-row processing of query results.

## Why Use Cursors?

- Process very large result sets without loading all into memory
- Navigate forward and backward through results
- Share query results between functions

## Declaring Cursors

```plpgsql
DECLARE
    -- Unbound cursor (query defined when opened)
    cur REFCURSOR;
    
    -- Bound cursor (query defined in declaration)
    user_cursor CURSOR FOR 
        SELECT * FROM users WHERE active = true;
    
    -- Parameterized cursor
    order_cursor CURSOR (user_id INTEGER) FOR
        SELECT * FROM orders WHERE user_id = $1;
```

## Opening Cursors

```plpgsql
-- Open unbound cursor
OPEN cur FOR SELECT * FROM products;

-- Open bound cursor
OPEN user_cursor;

-- Open parameterized cursor
OPEN order_cursor(42);

-- With dynamic query
OPEN cur FOR EXECUTE 'SELECT * FROM ' || quote_ident(table_name);
```

## Fetching Data

```plpgsql
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

## Fetch Directions

```plpgsql
-- Various fetch options (requires SCROLL cursor)
FETCH NEXT FROM cur INTO rec;      -- Next row (default)
FETCH PRIOR FROM cur INTO rec;     -- Previous row
FETCH FIRST FROM cur INTO rec;     -- First row
FETCH LAST FROM cur INTO rec;      -- Last row
FETCH ABSOLUTE 5 FROM cur INTO rec; -- 5th row
FETCH RELATIVE -2 FROM cur INTO rec; -- 2 rows before current
```

## SCROLL Cursors

```plpgsql
DECLARE
    -- Allow backward movement
    scroll_cur SCROLL CURSOR FOR 
        SELECT * FROM products;
    
    -- Prevent backward movement (default)
    no_scroll_cur NO SCROLL CURSOR FOR 
        SELECT * FROM products;
```

## Returning Cursors (Portal Pattern)

```plpgsql
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
SELECT get_users_cursor();  -- Returns cursor name
FETCH 10 FROM "<cursor_name>";
FETCH 10 FROM "<cursor_name>";
COMMIT;  -- Cursor closed
```

## FOR Loop with Cursors

```plpgsql
DECLARE
    user_cursor CURSOR FOR SELECT * FROM users;
    rec RECORD;
BEGIN
    -- Implicit open, fetch, close
    FOR rec IN user_cursor LOOP
        RAISE NOTICE 'User: %', rec.name;
    END LOOP;
END;
```

📖 [Cursors](https://www.postgresql.org/docs/18/plpgsql-cursors.html)

## Resources

- [Cursors](https://www.postgresql.org/docs/18/plpgsql-cursors.html) — Cursor documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*