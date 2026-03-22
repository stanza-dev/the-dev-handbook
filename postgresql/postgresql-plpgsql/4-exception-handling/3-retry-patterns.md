---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-retry-patterns"
---

# Error Patterns and Retry Logic

## Introduction

Not all database errors are permanent. Serialization failures, deadlocks, and unique-constraint races can be retried successfully. This lesson covers practical error-handling patterns that go beyond basic EXCEPTION blocks: retry loops for transient errors, upsert patterns that avoid exceptions entirely, and custom SQLSTATE codes for domain-specific error signaling.

## Key Concepts

- **Transient error**: An error caused by timing (deadlock, serialization failure) that may succeed on retry.
- **Retry loop**: A pattern that catches a transient error and re-executes the operation, typically with a maximum attempt count.
- **Upsert (ON CONFLICT)**: An INSERT ... ON CONFLICT DO UPDATE pattern that avoids unique_violation exceptions entirely.
- **Custom SQLSTATE**: Application-defined error codes (prefix 'P0' or 'XX') that let callers handle domain errors programmatically.

## Real World Context

A high-concurrency inventory system frequently encounters serialization failures when two transactions try to decrement the same product's stock simultaneously. Instead of failing and returning an error to the user, a retry loop inside a procedure catches the serialization failure and retries up to 3 times, making the operation transparent to the caller.

## Deep Dive

### Retry Loop for Transient Errors

```sql
CREATE PROCEDURE resilient_transfer(
    p_from_account INTEGER,
    p_to_account INTEGER,
    p_amount NUMERIC
) AS $$
DECLARE
    max_retries CONSTANT INTEGER := 3;
    attempt INTEGER := 0;
BEGIN
    LOOP
        attempt := attempt + 1;
        BEGIN
            UPDATE accounts SET balance = balance - p_amount WHERE id = p_from_account;
            UPDATE accounts SET balance = balance + p_amount WHERE id = p_to_account;
            COMMIT;
            RETURN;  -- Success, exit
        EXCEPTION
            WHEN serialization_failure OR deadlock_detected THEN
                IF attempt >= max_retries THEN
                    RAISE EXCEPTION 'Transfer failed after % attempts', max_retries
                        USING ERRCODE = 'serialization_failure';
                END IF;
                RAISE NOTICE 'Attempt % failed, retrying...', attempt;
                ROLLBACK;
        END;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

### Upsert to Avoid Exceptions

Instead of catching unique_violation, use ON CONFLICT:

```sql
CREATE FUNCTION upsert_user_preference(
    p_user_id INTEGER,
    p_key TEXT,
    p_value TEXT
) RETURNS VOID AS $$
BEGIN
    INSERT INTO user_preferences (user_id, key, value)
    VALUES (p_user_id, p_key, p_value)
    ON CONFLICT (user_id, key)
    DO UPDATE SET value = EXCLUDED.value, updated_at = NOW();
END;
$$ LANGUAGE plpgsql;
```

This is cleaner, faster, and avoids the subtransaction overhead of exception handling.

### Custom SQLSTATE Codes

Define domain-specific error codes for your application:

```sql
CREATE FUNCTION process_payment(p_order_id INTEGER, p_amount NUMERIC)
RETURNS VOID AS $$
DECLARE
    v_balance NUMERIC;
BEGIN
    SELECT balance INTO v_balance FROM wallets WHERE order_user_id = (
        SELECT user_id FROM orders WHERE id = p_order_id
    );

    IF v_balance < p_amount THEN
        RAISE EXCEPTION 'Insufficient funds: need %, have %', p_amount, v_balance
            USING ERRCODE = 'P0001',  -- Custom app error
                  HINT = 'Top up your wallet before retrying';
    END IF;

    -- Process the payment...
END;
$$ LANGUAGE plpgsql;
```

Callers can catch 'P0001' specifically and present a "top up wallet" UI flow.

## Common Pitfalls

1. **Retrying non-transient errors** -- Only retry serialization_failure and deadlock_detected. Retrying a check_violation or not_null_violation will fail every time.
2. **Missing the max-retry guard** -- A retry loop without a maximum attempt count can loop forever if the underlying issue is persistent.

## Best Practices

1. **Prefer ON CONFLICT over catching unique_violation** -- It is a single atomic statement with no subtransaction overhead.
2. **Use custom SQLSTATE codes starting with P0** -- PostgreSQL reserves this range for user-defined errors. It lets your API layer handle domain errors programmatically.

## Summary

- Retry loops catch transient errors (serialization_failure, deadlock_detected) and re-execute the operation up to a maximum number of attempts.
- ON CONFLICT (upsert) eliminates unique_violation exceptions entirely and is faster.
- Custom SQLSTATE codes (P0001, P0002, etc.) enable programmatic error handling in application code.

## Code Examples

**Uses ON CONFLICT to perform an upsert, which is faster and cleaner than catching unique_violation in an EXCEPTION block.**

```sql
-- Upsert pattern: avoids unique_violation exceptions
INSERT INTO user_preferences (user_id, key, value)
VALUES (42, 'theme', 'dark')
ON CONFLICT (user_id, key)
DO UPDATE SET value = EXCLUDED.value, updated_at = NOW();
```


## Resources

- [Error Codes](https://www.postgresql.org/docs/18/errcodes-appendix.html) — PostgreSQL error codes appendix

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*