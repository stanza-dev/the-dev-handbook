---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-trigger-basics"
---

# Trigger Fundamentals

## Introduction

Triggers automatically execute a function in response to data changes on a table. They are one of the most powerful features in PostgreSQL, enabling automatic audit logging, data validation, denormalized data maintenance, and complex business rules -- all enforced at the database level regardless of which application or service modifies the data.

## Key Concepts

- **Trigger**: A database object that fires a function automatically on INSERT, UPDATE, DELETE, or TRUNCATE events.
- **Trigger Function**: A function that returns TRIGGER and has access to special variables (NEW, OLD, TG_OP, etc.).
- **NEW**: The new row being inserted or the updated row (available in INSERT and UPDATE triggers).
- **OLD**: The existing row before modification (available in UPDATE and DELETE triggers).
- **TG_OP**: A string ('INSERT', 'UPDATE', 'DELETE', 'TRUNCATE') identifying the operation.
- **Row-level vs Statement-level**: Row-level triggers fire once per affected row; statement-level triggers fire once per SQL statement.

## Real World Context

An e-commerce platform needs to update a user's `order_count` and `total_spent` columns whenever an order is inserted, updated, or deleted. Without triggers, every microservice and admin tool that touches orders must remember to update these denormalized fields. A single trigger function guarantees consistency across all access paths.

## Deep Dive

### Trigger Anatomy

```sql
CREATE TRIGGER trigger_name
    {BEFORE | AFTER | INSTEAD OF}
    {INSERT | UPDATE | DELETE | TRUNCATE}
    ON table_name
    [FOR EACH {ROW | STATEMENT}]
    [WHEN (condition)]
    EXECUTE FUNCTION function_name();
```

### Trigger Function Requirements

```sql
CREATE FUNCTION trigger_function_name()
RETURNS TRIGGER AS $$
BEGIN
    -- trigger logic using NEW, OLD, TG_OP, etc.
    RETURN NEW;  -- or OLD, or NULL
END;
$$ LANGUAGE plpgsql;
```

### Special Variables

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

### Auto-Update Timestamp Example

```sql
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_column();
```

### BEFORE vs AFTER Triggers

**BEFORE triggers** can modify NEW values before they are saved and can return NULL to cancel the operation. Best for validation and data transformation.

**AFTER triggers** see the final data after constraints are checked but cannot modify the row. Best for audit logging and cascading changes.

**PostgreSQL 18 change**: AFTER triggers now run as the role that was active when the triggering event was queued, rather than the role active at commit time. This provides more predictable security behavior in complex transaction workflows.

### Row-Level vs Statement-Level

```sql
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

## Common Pitfalls

1. **Returning NULL from a BEFORE trigger** -- This silently cancels the INSERT/UPDATE/DELETE. Useful intentionally, but a common source of "my data disappeared" bugs when done accidentally.
2. **Accessing NEW in a DELETE trigger** -- NEW is NULL in DELETE triggers. Use OLD to access the row being deleted.

## Best Practices

1. **Keep trigger functions short and focused** -- A trigger that does too much makes debugging difficult and can cause unexpected performance issues.
2. **Use WHEN clauses to filter** -- Adding `WHEN (OLD.price IS DISTINCT FROM NEW.price)` avoids firing the trigger when the relevant column did not change.

## Summary

- Triggers fire automatically on INSERT, UPDATE, DELETE, or TRUNCATE and can run BEFORE, AFTER, or INSTEAD OF the operation.
- NEW and OLD provide access to the affected rows; TG_OP identifies the operation.
- In PostgreSQL 18, AFTER triggers run as the role active when events were queued, not at commit time.

## Code Examples

**A common trigger pattern that auto-updates an updated_at timestamp on every UPDATE, demonstrating BEFORE trigger, NEW record modification, and RETURN NEW.**

```sql
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_column();
```


## Resources

- [CREATE TRIGGER](https://www.postgresql.org/docs/18/sql-createtrigger.html) — Trigger syntax reference

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*