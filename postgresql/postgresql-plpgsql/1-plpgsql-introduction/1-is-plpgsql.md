---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-what-is-plpgsql"
---

# What is PL/pgSQL?

## Introduction

PL/pgSQL (Procedural Language/PostgreSQL) is PostgreSQL's native procedural programming language. It extends SQL with variables, loops, conditionals, and error handling, letting you write complex logic that runs directly inside the database engine. If you have ever wanted to keep business rules close to your data and eliminate unnecessary network round-trips, PL/pgSQL is the tool to learn.

## Key Concepts

- **PL/pgSQL**: A procedural language bundled with every PostgreSQL installation that augments SQL with control flow and variables.
- **Function**: A named PL/pgSQL block that returns a value and can be called inside SQL expressions.
- **Procedure**: A named block (since PostgreSQL 11) that can manage its own transactions and is invoked with the `CALL` statement.
- **Anonymous Block**: A one-off code block executed via the `DO` command without creating a persistent object.

## Real World Context

Imagine an e-commerce platform where every order must pass a multi-step validation: check inventory, calculate discounts, apply tax rules, and log an audit entry. Doing this in application code means multiple round-trips to the database. A single PL/pgSQL procedure can execute all those steps in one call, dramatically reducing latency and ensuring the rules are enforced consistently no matter which microservice touches the data.

## Deep Dive

### Why Use PL/pgSQL?

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

### Functions vs Procedures

**Functions** (since PostgreSQL 7.3):
- Return a value (scalar, table, or set)
- Can be called in SQL expressions
- Run inside caller's transaction

**Procedures** (since PostgreSQL 11):
- Don't necessarily return a value
- Can manage their own transactions (COMMIT/ROLLBACK)
- Called with CALL statement

The following example shows the syntactic difference between a function and a procedure:

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

Notice how the procedure calls `COMMIT` internally -- functions cannot do this.

### When NOT to Use PL/pgSQL

- Simple CRUD operations (use plain SQL)
- Application-specific UI logic
- Code that needs frequent changes and rapid deployment cycles
- Logic requiring external services or HTTP calls

Use PL/pgSQL for data-centric operations that benefit from proximity to the data.

## Common Pitfalls

1. **Overusing PL/pgSQL for simple queries** -- Writing a function that wraps a single `SELECT` adds overhead without benefit. Plain SQL is faster and easier to maintain.
2. **Confusing functions and procedures** -- Calling a procedure with `SELECT` instead of `CALL` (or vice versa) is a common beginner error that produces confusing error messages.

## Best Practices

1. **Start with plain SQL** -- Only reach for PL/pgSQL when SQL alone cannot express the logic you need.
2. **Prefer procedures for multi-step writes** -- Procedures can control transactions, making them ideal for batch operations that need intermediate commits.

## Summary

- PL/pgSQL is PostgreSQL's native procedural language for writing server-side logic with variables, loops, and error handling.
- Functions return values and run inside the caller's transaction; procedures can manage their own transactions.
- Use PL/pgSQL when complex business rules, audit logging, or batch processing benefit from running close to the data.

## Code Examples

**Demonstrates the key difference between a PL/pgSQL function (returns a value, callable in SQL) and a procedure (manages its own transaction, called with CALL).**

```sql
-- Function: returns a value
CREATE FUNCTION get_total(order_id INT) RETURNS NUMERIC AS $$
    SELECT SUM(quantity * price) FROM order_items WHERE order_id = $1;
$$ LANGUAGE sql;

SELECT order_id, get_total(order_id) FROM orders;

-- Procedure: performs action with transaction control
CREATE PROCEDURE archive_old_orders() AS $$
BEGIN
    INSERT INTO archived_orders SELECT * FROM orders WHERE created_at < NOW() - INTERVAL '1 year';
    DELETE FROM orders WHERE created_at < NOW() - INTERVAL '1 year';
    COMMIT;
END;
$$ LANGUAGE plpgsql;

CALL archive_old_orders();
```


## Resources

- [PL/pgSQL Overview](https://www.postgresql.org/docs/18/plpgsql.html) — Complete PL/pgSQL documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*