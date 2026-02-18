---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-exception-blocks"
---

# Exception Handling Blocks

Catch and handle errors gracefully.

## Basic Syntax

```plpgsql
BEGIN
    -- statements that might fail
EXCEPTION
    WHEN condition THEN
        -- handler statements
    WHEN condition2 THEN
        -- another handler
END;
```

## Common Exception Handlers

```plpgsql
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

## Multiple Conditions

```plpgsql
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

## WHEN OTHERS (Catch-All)

```plpgsql
BEGIN
    -- risky operations
EXCEPTION
    WHEN unique_violation THEN
        -- specific handling
        RETURN -1;
    WHEN OTHERS THEN
        -- catch everything else
        RAISE NOTICE 'Unexpected error: % %', SQLSTATE, SQLERRM;
        RETURN -999;
END;
```

## Getting Error Details

```plpgsql
CREATE FUNCTION safe_operation() RETURNS BOOLEAN AS $$
DECLARE
    err_context TEXT;
    err_message TEXT;
    err_detail TEXT;
    err_hint TEXT;
BEGIN
    -- operation that might fail
    PERFORM risky_function();
    RETURN TRUE;
    
EXCEPTION WHEN OTHERS THEN
    GET STACKED DIAGNOSTICS
        err_message = MESSAGE_TEXT,
        err_detail = PG_EXCEPTION_DETAIL,
        err_hint = PG_EXCEPTION_HINT,
        err_context = PG_EXCEPTION_CONTEXT;
    
    -- Log the error
    INSERT INTO error_log (sqlstate, message, detail, hint, context)
    VALUES (SQLSTATE, err_message, err_detail, err_hint, err_context);
    
    RETURN FALSE;
END;
$$ LANGUAGE plpgsql;
```

## Transaction Behavior

**Important**: Exception handlers roll back the block's changes:

```plpgsql
BEGIN
    INSERT INTO audit_log VALUES ('Starting');  -- This gets rolled back!
    INSERT INTO users (email) VALUES ('duplicate@test.com');
EXCEPTION
    WHEN unique_violation THEN
        -- The first INSERT is also rolled back
        RAISE NOTICE 'Both inserts rolled back';
END;
```

## Nested Blocks for Partial Rollback

```plpgsql
BEGIN
    INSERT INTO audit_log VALUES ('Starting');  -- Preserved
    
    BEGIN  -- Nested block
        INSERT INTO users (email) VALUES ('duplicate@test.com');
    EXCEPTION
        WHEN unique_violation THEN
            -- Only the nested block is rolled back
            RAISE NOTICE 'User insert failed, audit preserved';
    END;
    
    INSERT INTO audit_log VALUES ('Finished');  -- Preserved
END;
```

📖 [Trapping Errors](https://www.postgresql.org/docs/18/plpgsql-control-structures.html#PLPGSQL-ERROR-TRAPPING)

## Resources

- [Error Trapping](https://www.postgresql.org/docs/18/plpgsql-control-structures.html#PLPGSQL-ERROR-TRAPPING) — Exception handling documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*