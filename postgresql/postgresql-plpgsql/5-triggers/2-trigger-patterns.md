---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-trigger-patterns"
---

# Common Trigger Patterns

## Introduction

Triggers shine in recurring patterns: audit trails, data validation, denormalized counters, and change prevention. Rather than reinventing these each time, learning the standard patterns lets you implement them quickly and correctly. This lesson covers the most common real-world trigger patterns with production-ready examples.

## Key Concepts

- **Audit trigger**: Records old and new row data (as JSONB) for every change to a table.
- **Validation trigger**: A BEFORE trigger that checks business rules and either modifies data or raises an exception.
- **Denormalization trigger**: An AFTER trigger that maintains derived/cached data in another table.
- **Prevention trigger**: A BEFORE trigger that returns NULL or raises an exception to block certain operations.

## Real World Context

A healthcare application must log every change to patient records for compliance. A generic audit trigger attached to all sensitive tables captures the before/after state as JSONB, creating an immutable change history that auditors can query. Meanwhile, a prevention trigger on the patients table blocks deletion of records that are under active treatment.

## Deep Dive

### Audit Trail

```sql
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    operation TEXT NOT NULL,
    old_data JSONB,
    new_data JSONB,
    changed_by TEXT DEFAULT current_user,
    changed_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION audit_trigger_func()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, old_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(OLD));
        RETURN OLD;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, old_data, new_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(OLD), to_jsonb(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, new_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(NEW));
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_audit
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_func();
```

### Data Validation

```sql
CREATE OR REPLACE FUNCTION validate_order()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.total <= 0 THEN
        RAISE EXCEPTION 'Order total must be positive';
    END IF;

    IF TG_OP = 'INSERT' THEN
        NEW.created_at := NOW();
        NEW.status := 'pending';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validate_order_trigger
    BEFORE INSERT OR UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION validate_order();
```

### Maintaining Denormalized Counts

```sql
CREATE OR REPLACE FUNCTION update_order_count()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE users SET order_count = order_count + 1 WHERE id = NEW.user_id;
    ELSIF TG_OP = 'DELETE' THEN
        UPDATE users SET order_count = order_count - 1 WHERE id = OLD.user_id;
    ELSIF TG_OP = 'UPDATE' AND OLD.user_id != NEW.user_id THEN
        UPDATE users SET order_count = order_count - 1 WHERE id = OLD.user_id;
        UPDATE users SET order_count = order_count + 1 WHERE id = NEW.user_id;
    END IF;
    RETURN NULL;  -- AFTER trigger, return value ignored
END;
$$ LANGUAGE plpgsql;
```

### Preventing Changes

```sql
CREATE OR REPLACE FUNCTION prevent_delete_admin()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.role = 'admin' THEN
        RAISE EXCEPTION 'Cannot delete admin users';
    END IF;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;
```

### Conditional Triggers (WHEN clause)

```sql
CREATE TRIGGER log_price_change
    AFTER UPDATE OF price ON products
    FOR EACH ROW
    WHEN (OLD.price IS DISTINCT FROM NEW.price
          AND ABS(NEW.price - OLD.price) > 10)
    EXECUTE FUNCTION log_significant_change();
```

### PostgreSQL 18: OLD/NEW in RETURNING

PostgreSQL 18 introduces the ability to reference OLD and NEW in RETURNING clauses of DML statements, making it easier to see both the before and after state in a single query:

```sql
-- See both old and new price in one UPDATE statement
UPDATE products
SET price = price * 1.1
WHERE category = 'electronics'
RETURNING id, OLD.price AS old_price, NEW.price AS new_price;
```

This reduces the need for triggers that simply capture before/after values.

## Common Pitfalls

1. **Trigger recursion** -- A trigger on table A that modifies table A can fire itself recursively. Use `pg_trigger_depth()` to detect and break recursion.
2. **Deadlocks from cross-table triggers** -- An AFTER trigger on table A updating table B, combined with a trigger on B updating A, can deadlock under concurrent writes.

## Best Practices

1. **Use generic trigger functions** -- A single audit function can serve multiple tables, reducing maintenance.
2. **Use IS DISTINCT FROM for change detection** -- It correctly handles NULL-to-value and value-to-NULL transitions that `!=` misses.

## Summary

- Audit triggers capture before/after data as JSONB for change history.
- Validation triggers enforce business rules; prevention triggers block dangerous operations.
- PostgreSQL 18 adds OLD/NEW in RETURNING clauses, reducing the need for some trigger patterns.

## Code Examples

**A generic audit trigger function that logs INSERT, UPDATE, and DELETE operations with before/after data as JSONB.**

```sql
CREATE OR REPLACE FUNCTION audit_trigger_func()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, old_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(OLD));
        RETURN OLD;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, old_data, new_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(OLD), to_jsonb(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, new_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(NEW));
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;
```


## Resources

- [Trigger Functions](https://www.postgresql.org/docs/18/plpgsql-trigger.html) — Writing trigger functions

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*