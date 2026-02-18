---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-what-is-plpgsql"
---

# What is PL/pgSQL?

PL/pgSQL (Procedural Language/PostgreSQL) is PostgreSQL's native procedural programming language. It extends SQL with variables, loops, conditionals, and error handling.

## Why Use PL/pgSQL?

**Performance Benefits:**
- Reduce network round-trips between application and database
- Execute complex operations in a single call
- Keep data processing close to the data

**Business Logic Benefits:**
- Enforce complex rules at the database level
- Ensure consistency regardless of which application accesses data
- Implement audit logging automatically

**Use Cases:**
- Complex data validation
- Audit trails and history tracking
- Calculated/derived fields
- Data migration and transformation
- Batch processing operations

## Functions vs Procedures

**Functions** (since PostgreSQL 7.3):
- Return a value (scalar, table, or set)
- Can be called in SQL expressions
- Run inside caller's transaction

**Procedures** (since PostgreSQL 11):
- Don't necessarily return a value
- Can manage their own transactions (COMMIT/ROLLBACK)
- Called with CALL statement

```sql
-- Function: returns a value
CREATE FUNCTION get_total(order_id INT) RETURNS NUMERIC AS $$
    SELECT SUM(quantity * price) FROM order_items WHERE order_id = $1;
$$ LANGUAGE sql;

-- Usage in queries
SELECT order_id, get_total(order_id) FROM orders;

-- Procedure: performs action
CREATE PROCEDURE archive_old_orders() AS $$
BEGIN
    INSERT INTO archived_orders SELECT * FROM orders WHERE created_at < NOW() - INTERVAL '1 year';
    DELETE FROM orders WHERE created_at < NOW() - INTERVAL '1 year';
    COMMIT;  -- Can commit inside procedure!
END;
$$ LANGUAGE plpgsql;

-- Usage
CALL archive_old_orders();
```

## When NOT to Use PL/pgSQL

❌ Simple CRUD operations (use plain SQL)
❌ Application-specific UI logic
❌ Code that needs frequent changes
❌ Logic requiring external services

✅ Use PL/pgSQL for data-centric operations that benefit from proximity to the data.

📖 [PL/pgSQL Overview](https://www.postgresql.org/docs/18/plpgsql-overview.html)

## Resources

- [PL/pgSQL Overview](https://www.postgresql.org/docs/18/plpgsql.html) — Complete PL/pgSQL documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*