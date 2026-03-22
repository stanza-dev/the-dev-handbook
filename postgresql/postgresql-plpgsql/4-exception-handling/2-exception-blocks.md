---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-exception-blocks"
---

# Exception Handling Blocks

## Introduction

Exception handling lets your PL/pgSQL code catch errors and recover gracefully instead of crashing the entire transaction. Understanding how BEGIN/EXCEPTION/END blocks work -- including their automatic subtransaction rollback behavior -- is essential for writing resilient database functions.

## Key Concepts

- **EXCEPTION block**: A section within BEGIN...END that catches errors matching specified conditions.
- **SQLERRM**: A special variable containing the error message text inside an exception handler.
- **SQLSTATE**: A special variable containing the 5-character error code inside an exception handler.
- **WHEN OTHERS**: A catch-all handler that matches any exception not handled by previous WHEN clauses.
- **Subtransaction rollback**: When an exception is caught, all database changes made within that block are automatically rolled back.
- **GET STACKED DIAGNOSTICS**: Retrieves detailed error metadata (message, detail, hint, context) inside exception handlers.

## Real World Context

A user registration function attempts an INSERT that may violate a unique constraint on email. Without exception handling, the entire transaction fails and the caller gets a raw PostgreSQL error. With a targeted WHEN unique_violation handler, you can return a friendly error and let the rest of the transaction continue.

## Deep Dive

### Basic Syntax

```sql
BEGIN
    -- statements that might fail
EXCEPTION
    WHEN condition THEN
        -- handler statements
    WHEN condition2 THEN
        -- another handler
END;
```

### Common Exception Handlers

```sql
CREATE FUNCTION safe_divide(a NUMERIC, b NUMERIC) RETURNS NUMERIC AS $$
BEGIN
    RETURN a / b;
EXCEPTION
    WHEN division_by_zero THEN
        RAISE NOTICE 'Division by zero, returning NULL';
        RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

### Multiple Conditions

```sql
BEGIN
    INSERT INTO users (email) VALUES (input_email);
EXCEPTION
    WHEN unique_violation THEN
        RAISE EXCEPTION 'Email already exists';
    WHEN not_null_violation THEN
        RAISE EXCEPTION 'Email cannot be empty';
    WHEN check_violation THEN
        RAISE EXCEPTION 'Invalid email format';
END;
```

### Getting Error Details

```sql
CREATE FUNCTION safe_operation() RETURNS BOOLEAN AS $$
DECLARE
    err_message TEXT;
    err_detail TEXT;
    err_hint TEXT;
    err_context TEXT;
BEGIN
    PERFORM risky_function();
    RETURN TRUE;
EXCEPTION WHEN OTHERS THEN
    GET STACKED DIAGNOSTICS
        err_message = MESSAGE_TEXT,
        err_detail = PG_EXCEPTION_DETAIL,
        err_hint = PG_EXCEPTION_HINT,
        err_context = PG_EXCEPTION_CONTEXT;

    INSERT INTO error_log (sqlstate, message, detail, hint, context)
    VALUES (SQLSTATE, err_message, err_detail, err_hint, err_context);

    RETURN FALSE;
END;
$$ LANGUAGE plpgsql;
```

### Transaction Behavior and Nested Blocks

Exception handlers roll back all changes within the block. Use nested blocks for partial rollback:

```sql
BEGIN
    INSERT INTO audit_log VALUES ('Starting');  -- Preserved

    BEGIN  -- Nested block
        INSERT INTO users (email) VALUES ('duplicate@test.com');
    EXCEPTION
        WHEN unique_violation THEN
            -- Only the nested block's INSERT is rolled back
            RAISE NOTICE 'User insert failed, audit preserved';
    END;

    INSERT INTO audit_log VALUES ('Finished');  -- Preserved
END;
```

## Common Pitfalls

1. **Using WHEN OTHERS without re-raising** -- Silently catching all exceptions hides bugs. Always log the error and consider re-raising after logging.
2. **Forgetting that exception blocks roll back the entire block** -- If you INSERT into a log table and then hit an error in the same block, the log INSERT is also rolled back. Use nested blocks to isolate side effects.

## Best Practices

1. **Catch specific exceptions first, use WHEN OTHERS as a last resort** -- Specific handlers give you precise control; WHEN OTHERS should log and re-raise.
2. **Use nested blocks to control rollback scope** -- Wrap only the risky operation in its own block so that surrounding work is preserved on failure.

## Summary

- BEGIN/EXCEPTION/END blocks catch errors and let your code recover instead of crashing the transaction.
- SQLERRM and SQLSTATE provide error details; GET STACKED DIAGNOSTICS gives full context.
- Exception handlers roll back all changes in their block, so use nested blocks for fine-grained rollback control.

## Code Examples

**Shows nested blocks for partial rollback: the outer audit_log inserts survive even when the inner user insert fails with a unique violation.**

```sql
BEGIN
    INSERT INTO audit_log VALUES ('Starting');  -- Preserved

    BEGIN  -- Nested block for isolation
        INSERT INTO users (email) VALUES ('dup@test.com');
    EXCEPTION
        WHEN unique_violation THEN
            RAISE NOTICE 'User insert failed, audit preserved';
    END;

    INSERT INTO audit_log VALUES ('Finished');  -- Preserved
END;
```


## Resources

- [Error Trapping](https://www.postgresql.org/docs/18/plpgsql-control-structures.html#PLPGSQL-ERROR-TRAPPING) — Exception handling documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*