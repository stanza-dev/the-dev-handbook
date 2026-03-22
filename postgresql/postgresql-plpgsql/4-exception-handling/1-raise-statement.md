---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-raise-statement"
---

# The RAISE Statement

## Introduction

RAISE is how PL/pgSQL communicates with the outside world -- whether that means logging debug info, warning about deprecated usage, or aborting a transaction with a meaningful error. Understanding RAISE levels and the USING clause for structured error metadata will make your functions easier to debug and your error messages actionable for callers.

## Key Concepts

- **RAISE levels**: DEBUG, LOG, INFO, NOTICE, WARNING (all non-fatal messages) and EXCEPTION (aborts the current transaction).
- **Format strings**: Use `%` as a placeholder in RAISE messages, replaced left-to-right with arguments.
- **USING clause**: Attaches structured metadata to a RAISE -- ERRCODE, HINT, DETAIL, COLUMN, CONSTRAINT, etc.
- **SQLSTATE**: A 5-character error code (e.g., '23505' for unique_violation). Can be used as the ERRCODE.

## Real World Context

When a payment processing function rejects a transaction, a bare "Payment failed" message forces the caller to guess what went wrong. Using RAISE EXCEPTION with ERRCODE, HINT, and DETAIL lets the API layer return a structured error response: "duplicate payment (SQLSTATE 23505), hint: check for a recent payment with this transaction ID."

## Deep Dive

### Message Levels

```sql
RAISE DEBUG 'Variable x = %', x;        -- Hidden by default
RAISE LOG 'Function called with %', p;   -- Server log only
RAISE NOTICE 'Processing row %', n;      -- Shown in psql
RAISE WARNING 'Deprecated function';      -- Warning to client
RAISE EXCEPTION 'Invalid ID: %', id;     -- Aborts transaction
```

### Format Strings

```sql
RAISE NOTICE 'User % has % orders', user_name, order_count;
RAISE NOTICE 'Discount is 15%% off';  -- Use %% to escape %
```

### USING Clause Options

```sql
RAISE EXCEPTION 'Duplicate email'
    USING ERRCODE = 'unique_violation';

RAISE EXCEPTION 'Order validation failed'
    USING
        HINT = 'Check that all items are in stock',
        DETAIL = 'Product ID 42 has insufficient quantity',
        ERRCODE = 'check_violation';

-- Re-raise using condition name
RAISE unique_violation USING MESSAGE = 'Custom message';
```

### Re-raising Exceptions

```sql
BEGIN
    -- some operations
EXCEPTION WHEN OTHERS THEN
    INSERT INTO error_log (message) VALUES (SQLERRM);
    RAISE;  -- Re-raise the original exception
END;
```

### Debugging with RAISE

```sql
CREATE FUNCTION debug_example(input_val INTEGER) RETURNS INTEGER AS $$
DECLARE
    step INTEGER := 0;
BEGIN
    step := 1;
    RAISE NOTICE 'Step %: Input is %', step, input_val;

    step := 2;
    IF input_val < 0 THEN
        RAISE NOTICE 'Step %: Converting negative', step;
        input_val := ABS(input_val);
    END IF;

    step := 3;
    RAISE NOTICE 'Step %: Returning %', step, input_val * 2;
    RETURN input_val * 2;
END;
$$ LANGUAGE plpgsql;
```

## Common Pitfalls

1. **Using EXCEPTION level for non-fatal messages** -- RAISE EXCEPTION aborts the transaction. Use NOTICE or WARNING for informational messages.
2. **Forgetting to escape % in messages** -- A literal percent sign requires `%%`. A bare `%` without a matching argument causes a runtime error.

## Best Practices

1. **Always provide an ERRCODE with EXCEPTION** -- It lets callers handle errors programmatically instead of parsing message strings.
2. **Add HINT and DETAIL for user-facing errors** -- These fields appear in error responses and help developers diagnose issues without reading function source code.

## Summary

- RAISE sends messages at six severity levels, with only EXCEPTION aborting the transaction.
- Format strings use `%` for argument substitution and `%%` for literal percent signs.
- The USING clause attaches ERRCODE, HINT, and DETAIL to give callers structured, actionable error information.

## Code Examples

**Demonstrates RAISE EXCEPTION with the USING clause to provide structured error metadata including ERRCODE, HINT, and DETAIL.**

```sql
RAISE EXCEPTION 'Order validation failed'
    USING
        HINT = 'Check that all items are in stock',
        DETAIL = 'Product ID 42 has insufficient quantity',
        ERRCODE = 'check_violation';
```


## Resources

- [Errors and Messages](https://www.postgresql.org/docs/18/plpgsql-errors-and-messages.html) — RAISE statement documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*