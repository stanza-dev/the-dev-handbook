---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-return-null"
---

# RETURN, NULL Handling, and Early Exit

## Introduction

PL/pgSQL functions often need to exit early when input is invalid, return different types of values, or handle NULL gracefully. Understanding the RETURN statement's variants and combining them with NULL-safe operators will help you write functions that are both correct and defensive.

## Key Concepts

- **RETURN**: Exits the function immediately and provides the return value.
- **RETURN NEXT**: In SETOF functions, adds a row to the result set without exiting.
- **RETURN QUERY**: In SETOF functions, appends an entire query result to the output.
- **IS DISTINCT FROM**: A NULL-safe inequality operator -- `NULL IS DISTINCT FROM NULL` is FALSE, unlike `NULL != NULL` which is NULL.
- **COALESCE**: Returns the first non-NULL argument, useful for providing defaults.

## Real World Context

A function that calculates a user's loyalty discount might receive NULL for the membership tier (new users without a tier). Without a guard clause, the NULL propagates through arithmetic and the function silently returns NULL, which downstream code interprets as "no discount" or, worse, causes a crash. A guard clause with `RETURN 0` at the top makes the behavior explicit.

## Deep Dive

### Guard Clauses with Early RETURN

The guard clause pattern validates inputs at the top and exits immediately if anything is wrong:

```sql
CREATE FUNCTION calculate_loyalty_discount(
    p_user_id INTEGER,
    p_order_total NUMERIC
) RETURNS NUMERIC AS $$
DECLARE
    v_tier TEXT;
    v_discount NUMERIC;
BEGIN
    -- Guard: reject NULL input
    IF p_user_id IS NULL OR p_order_total IS NULL THEN
        RETURN 0;
    END IF;

    -- Guard: reject invalid total
    IF p_order_total <= 0 THEN
        RETURN 0;
    END IF;

    SELECT membership_tier INTO v_tier
    FROM users WHERE id = p_user_id;

    IF NOT FOUND THEN
        RETURN 0;  -- User not found, no discount
    END IF;

    CASE v_tier
        WHEN 'gold' THEN v_discount := p_order_total * 0.15;
        WHEN 'silver' THEN v_discount := p_order_total * 0.10;
        WHEN 'bronze' THEN v_discount := p_order_total * 0.05;
        ELSE v_discount := 0;
    END CASE;

    RETURN v_discount;
END;
$$ LANGUAGE plpgsql;
```

### NULL-Safe Comparisons

```sql
-- Standard comparison: NULL = NULL is NULL (falsy)
IF old_value = new_value THEN ...  -- Fails for NULLs

-- NULL-safe: IS NOT DISTINCT FROM treats NULLs as equal
IF old_value IS NOT DISTINCT FROM new_value THEN
    -- true when both are NULL or both are the same value
END IF;

-- COALESCE for defaults
v_name := COALESCE(input_name, 'Anonymous');
v_total := COALESCE(subtotal, 0) + COALESCE(tax, 0);
```

### RETURN in VOID Functions

For procedures or VOID functions, a bare RETURN exits without a value:

```sql
CREATE FUNCTION log_if_important(p_level TEXT, p_msg TEXT)
RETURNS VOID AS $$
BEGIN
    IF p_level NOT IN ('WARNING', 'EXCEPTION') THEN
        RETURN;  -- Exit early, nothing to log
    END IF;
    INSERT INTO important_logs (level, message) VALUES (p_level, p_msg);
END;
$$ LANGUAGE plpgsql;
```

## Common Pitfalls

1. **NULL propagation through arithmetic** -- `NULL + 10` is NULL, not 10. Always wrap potentially-NULL values with COALESCE before doing math.
2. **Using `!=` to compare NULLable columns** -- `NULL != 'x'` evaluates to NULL, not TRUE. Use `IS DISTINCT FROM` for NULL-safe inequality.

## Best Practices

1. **Use guard clauses at the top of functions** -- Validate inputs early and RETURN immediately to keep the main logic path clean and un-nested.
2. **Use IS DISTINCT FROM in triggers** -- When comparing OLD and NEW values to detect actual changes, this operator correctly handles NULL-to-value transitions.

## Summary

- RETURN exits a function immediately; use guard clauses for early validation.
- COALESCE and IS DISTINCT FROM are essential tools for NULL-safe PL/pgSQL code.
- In SETOF functions, RETURN NEXT and RETURN QUERY add rows without exiting; a final bare RETURN signals completion.

## Code Examples

**Guard clause pattern with early RETURN for NULL inputs and unknown users, combined with a CASE expression for tier-based discounts.**

```sql
CREATE FUNCTION calculate_loyalty_discount(
    p_user_id INTEGER,
    p_order_total NUMERIC
) RETURNS NUMERIC AS $$
DECLARE
    v_tier TEXT;
BEGIN
    IF p_user_id IS NULL OR p_order_total IS NULL THEN
        RETURN 0;  -- Guard clause
    END IF;

    SELECT membership_tier INTO v_tier FROM users WHERE id = p_user_id;

    IF NOT FOUND THEN RETURN 0; END IF;

    RETURN CASE v_tier
        WHEN 'gold' THEN p_order_total * 0.15
        WHEN 'silver' THEN p_order_total * 0.10
        ELSE 0
    END;
END;
$$ LANGUAGE plpgsql;
```


## Resources

- [PL/pgSQL Control Structures](https://www.postgresql.org/docs/18/plpgsql-control-structures.html) — RETURN statements and control flow

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*