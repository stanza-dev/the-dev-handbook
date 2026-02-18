---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-raise-statement"
---

# The RAISE Statement

RAISE reports messages and errors from PL/pgSQL code.

## Message Levels

```plpgsql
-- DEBUG: Detailed debugging info (usually hidden)
RAISE DEBUG 'Variable x = %', x;

-- LOG: Server log only
RAISE LOG 'Function called with param %', param;

-- INFO: Informational (not shown by default)
RAISE INFO 'Processing started';

-- NOTICE: Important info (shown by default in psql)
RAISE NOTICE 'Processing row %', row_number;

-- WARNING: Warning message
RAISE WARNING 'Deprecated function called';

-- EXCEPTION: Aborts transaction
RAISE EXCEPTION 'Invalid user ID: %', user_id;
```

## Format Strings

```plpgsql
-- Use % as placeholder
RAISE NOTICE 'User % has % orders', user_name, order_count;

-- Multiple placeholders
RAISE NOTICE 'Processing: id=%, name=%, email=%', id, name, email;

-- Escape % with %%
RAISE NOTICE 'Discount is 15%% off';
```

## USING Clause Options

```plpgsql
-- Custom error code
RAISE EXCEPTION 'Duplicate email' 
    USING ERRCODE = 'unique_violation';

-- Or use SQLSTATE directly
RAISE EXCEPTION 'Duplicate email' 
    USING ERRCODE = '23505';

-- Additional context
RAISE EXCEPTION 'Order validation failed'
    USING 
        HINT = 'Check that all items are in stock',
        DETAIL = 'Product ID 42 has insufficient quantity',
        ERRCODE = 'check_violation';

-- Using condition name
RAISE division_by_zero;
RAISE unique_violation USING MESSAGE = 'Custom message';
```

## Re-raising Exceptions

```plpgsql
BEGIN
    -- some operations
EXCEPTION WHEN OTHERS THEN
    -- Log the error
    INSERT INTO error_log (message) VALUES (SQLERRM);
    -- Re-raise the original exception
    RAISE;
END;
```

## Debugging with RAISE

```plpgsql
CREATE FUNCTION debug_example(input_val INTEGER) RETURNS INTEGER AS $$
DECLARE
    step INTEGER := 0;
BEGIN
    step := 1;
    RAISE NOTICE 'Step %: Input value is %', step, input_val;
    
    step := 2;
    IF input_val < 0 THEN
        RAISE NOTICE 'Step %: Converting negative to positive', step;
        input_val := ABS(input_val);
    END IF;
    
    step := 3;
    RAISE NOTICE 'Step %: Returning %', step, input_val * 2;
    
    RETURN input_val * 2;
END;
$$ LANGUAGE plpgsql;
```

📖 [Errors and Messages](https://www.postgresql.org/docs/18/plpgsql-errors-and-messages.html)

## Resources

- [Errors and Messages](https://www.postgresql.org/docs/18/plpgsql-errors-and-messages.html) — RAISE statement documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*