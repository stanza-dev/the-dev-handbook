---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-dynamic-sql"
---

# Dynamic SQL with EXECUTE

Build and execute SQL statements at runtime.

## Basic EXECUTE

```plpgsql
EXECUTE 'SELECT COUNT(*) FROM users' INTO user_count;

EXECUTE 'INSERT INTO logs (message) VALUES ($1)' USING 'Hello';
```

## Building Queries Dynamically

```plpgsql
CREATE FUNCTION search_table(
    table_name TEXT,
    column_name TEXT,
    search_value TEXT
) RETURNS SETOF RECORD AS $$
DECLARE
    query TEXT;
BEGIN
    -- Build query dynamically
    query := 'SELECT * FROM ' || quote_ident(table_name) ||
             ' WHERE ' || quote_ident(column_name) || ' = $1';
    
    RETURN QUERY EXECUTE query USING search_value;
END;
$$ LANGUAGE plpgsql;
```

## Security: SQL Injection Prevention

**DANGER - Never do this:**
```plpgsql
-- VULNERABLE TO SQL INJECTION!
query := 'SELECT * FROM users WHERE name = ''' || user_input || '''';
```

**Safe approaches:**

### 1. Use USING clause (preferred)
```plpgsql
EXECUTE 'SELECT * FROM users WHERE name = $1' USING user_input;
```

### 2. Use quote_literal() for values
```plpgsql
query := 'SELECT * FROM users WHERE name = ' || quote_literal(user_input);
```

### 3. Use quote_ident() for identifiers
```plpgsql
query := 'SELECT * FROM ' || quote_ident(table_name);
```

### 4. Use format() function (cleanest)
```plpgsql
-- %I = identifier (quoted), %L = literal (quoted), %s = simple string
query := format('SELECT * FROM %I WHERE %I = %L', 
                table_name, column_name, search_value);
```

## Complete Example: Dynamic Table Cleanup

```plpgsql
CREATE FUNCTION cleanup_old_records(
    target_table TEXT,
    date_column TEXT,
    days_old INTEGER
) RETURNS INTEGER AS $$
DECLARE
    deleted_count INTEGER;
    cutoff_date DATE;
BEGIN
    cutoff_date := CURRENT_DATE - days_old;
    
    EXECUTE format(
        'DELETE FROM %I WHERE %I < $1',
        target_table,
        date_column
    ) USING cutoff_date;
    
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    
    RAISE NOTICE 'Deleted % rows from % older than %', 
                 deleted_count, target_table, cutoff_date;
    
    RETURN deleted_count;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT cleanup_old_records('audit_logs', 'created_at', 90);
```

## When to Use Dynamic SQL

✅ **Use when:**
- Table or column names are parameters
- Building complex conditional queries
- Generic utility functions

❌ **Avoid when:**
- Query structure is fixed (use plain SQL)
- Only values change (use parameterized queries)

📖 [Executing Dynamic Commands](https://www.postgresql.org/docs/18/plpgsql-statements.html#PLPGSQL-STATEMENTS-EXECUTING-DYN)

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*