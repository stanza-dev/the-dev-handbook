---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-trigger-basics"
---

# Trigger Fundamentals

Triggers automatically execute functions when data changes.

## Trigger Anatomy

```sql
CREATE TRIGGER trigger_name
    {BEFORE | AFTER | INSTEAD OF}
    {INSERT | UPDATE | DELETE | TRUNCATE}
    ON table_name
    [FOR EACH {ROW | STATEMENT}]
    [WHEN (condition)]
    EXECUTE FUNCTION function_name();
```

## Trigger Function Requirements

Trigger functions have a special signature:

```plpgsql
CREATE FUNCTION trigger_function_name()
RETURNS TRIGGER AS $$
BEGIN
    -- trigger logic
    RETURN NEW;  -- or OLD, or NULL
END;
$$ LANGUAGE plpgsql;
```

## Special Variables in Triggers

| Variable | Description |
|----------|-------------|
| `NEW` | New row (INSERT/UPDATE) |
| `OLD` | Old row (UPDATE/DELETE) |
| `TG_OP` | Operation: 'INSERT', 'UPDATE', 'DELETE', 'TRUNCATE' |
| `TG_NAME` | Trigger name |
| `TG_TABLE_NAME` | Table that fired trigger |
| `TG_TABLE_SCHEMA` | Schema of trigger table |
| `TG_WHEN` | 'BEFORE', 'AFTER', or 'INSTEAD OF' |
| `TG_LEVEL` | 'ROW' or 'STATEMENT' |

## Simple Example: Auto-Update Timestamp

```plpgsql
-- Trigger function
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER set_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_column();

-- Now every UPDATE automatically sets updated_at
UPDATE users SET name = 'New Name' WHERE id = 1;
-- updated_at is automatically set to NOW()
```

## BEFORE vs AFTER Triggers

**BEFORE triggers:**
- Can modify NEW values before they're saved
- Return NULL to cancel the operation
- Best for: validation, data transformation

**AFTER triggers:**
- See final data after constraints are checked
- Cannot modify the row
- Best for: audit logging, cascading changes

## Row-Level vs Statement-Level

```plpgsql
-- ROW level: fires once per affected row
CREATE TRIGGER per_row_trigger
    AFTER INSERT ON orders
    FOR EACH ROW
    EXECUTE FUNCTION log_new_order();

-- STATEMENT level: fires once per statement
CREATE TRIGGER per_statement_trigger
    AFTER INSERT ON orders
    FOR EACH STATEMENT
    EXECUTE FUNCTION summarize_inserts();
```

📖 [CREATE TRIGGER](https://www.postgresql.org/docs/18/sql-createtrigger.html)

## Resources

- [CREATE TRIGGER](https://www.postgresql.org/docs/18/sql-createtrigger.html) — Trigger syntax reference

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*