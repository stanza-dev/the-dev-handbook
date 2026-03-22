---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-dynamic-sql"
---

# Dynamic SQL with EXECUTE

## Introduction

Sometimes the table name, column name, or even the entire query structure is not known until runtime. PL/pgSQL's `EXECUTE` command lets you build and run SQL strings dynamically. This power comes with responsibility: dynamic SQL is the primary vector for SQL injection in database functions, so mastering safe parameterization is critical.

## Key Concepts

- **EXECUTE**: Runs a dynamically-constructed SQL string.
- **USING**: Passes parameter values to placeholders ($1, $2, ...) in the dynamic SQL string, preventing injection.
- **format()**: A string formatting function with `%I` (identifier, safely quoted), `%L` (literal, safely quoted), and `%s` (simple string, no quoting).
- **quote_ident()**: Safely quotes a string for use as an SQL identifier (table/column name).
- **quote_literal()**: Safely quotes a string for use as an SQL literal value.

## Real World Context

A multi-tenant SaaS application might store each tenant's data in a separate schema. A generic cleanup function needs to delete old records from `tenant_42.audit_logs`, where both the schema and table names come from parameters. Without proper quoting, a malicious tenant name could inject SQL. Using `format('%I.%I', schema_name, table_name)` ensures safety.

## Deep Dive

### Basic EXECUTE

```sql
EXECUTE 'SELECT COUNT(*) FROM users' INTO user_count;

EXECUTE 'INSERT INTO logs (message) VALUES ($1)' USING 'Hello';
```

### Security: SQL Injection Prevention

Never concatenate user input directly into SQL strings:

```sql
-- DANGER: vulnerable to SQL injection!
query := 'SELECT * FROM users WHERE name = ''' || user_input || '''';

-- SAFE: Use USING clause (preferred)
EXECUTE 'SELECT * FROM users WHERE name = $1' USING user_input;

-- SAFE: Use quote_literal() for values
query := 'SELECT * FROM users WHERE name = ' || quote_literal(user_input);

-- SAFE: Use quote_ident() for identifiers
query := 'SELECT * FROM ' || quote_ident(table_name);

-- SAFE: Use format() (cleanest)
query := format('SELECT * FROM %I WHERE %I = %L',
                table_name, column_name, search_value);
```

### Complete Example: Dynamic Table Cleanup

```sql
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
```

### When to Use Dynamic SQL

Use when: table or column names are parameters, building complex conditional queries, or writing generic utility functions.

Avoid when: query structure is fixed (use plain SQL) or only values change (use standard parameterized queries).

## Common Pitfalls

1. **String-concatenating user input** -- This is the number one cause of SQL injection in PL/pgSQL. Always use USING or format() with %I/%L.
2. **Using %s instead of %I for identifiers** -- `%s` does not add quotes, so a table name containing spaces or reserved words will break the query.

## Best Practices

1. **Use USING for values, format(%I) for identifiers** -- This combination covers virtually all safe dynamic SQL needs.
2. **Limit dynamic SQL to generic utilities** -- If you know the table and column at development time, write static SQL instead.

## Summary

- EXECUTE runs dynamically-constructed SQL strings and supports USING for safe parameter binding.
- format() with %I (identifiers) and %L (literals) is the cleanest way to build safe dynamic SQL.
- Only use dynamic SQL when the query structure genuinely varies at runtime; static SQL is simpler and faster.

## Code Examples

**A generic cleanup function using format(%I) for safe identifier quoting and USING for safe value binding in dynamic SQL.**

```sql
CREATE FUNCTION cleanup_old_records(
    target_table TEXT,
    date_column TEXT,
    days_old INTEGER
) RETURNS INTEGER AS $$
DECLARE
    deleted_count INTEGER;
BEGIN
    EXECUTE format(
        'DELETE FROM %I WHERE %I < $1',
        target_table, date_column
    ) USING CURRENT_DATE - days_old;

    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    RETURN deleted_count;
END;
$$ LANGUAGE plpgsql;
```


## Resources

- [Executing Dynamic Commands](https://www.postgresql.org/docs/18/plpgsql-statements.html#PLPGSQL-STATEMENTS-EXECUTING-DYN) — Dynamic SQL in PL/pgSQL

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*