---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-best-practices"
---

# Best Practices & Optimization

## Introduction

Writing PL/pgSQL that works is the first step; writing PL/pgSQL that is fast, secure, and maintainable is the goal. This lesson covers the patterns and settings that separate production-quality functions from quick prototypes: volatility categories, naming conventions, security modes, and the critical habit of preferring set-based SQL over procedural loops.

## Key Concepts

- **Volatility (IMMUTABLE/STABLE/VOLATILE)**: Tells the query planner how a function behaves, enabling optimizations.
- **SECURITY DEFINER**: The function runs with the privileges of its owner rather than the caller.
- **STRICT (on SELECT INTO)**: Raises an error if the query returns zero or more than one row.
- **Naming conventions**: Using prefixes like `p_` for parameters and `v_` for variables avoids column-name collisions.

## Real World Context

A function marked as VOLATILE (the default) cannot be used in index expressions and forces the planner to re-evaluate it for every row. Marking a pure calculation as IMMUTABLE lets PostgreSQL evaluate it once and reuse the result, dramatically speeding up queries that call it in WHERE clauses or use it in functional indexes.

## Deep Dive

### Naming Conventions

```sql
CREATE FUNCTION get_user(p_user_id INTEGER)
RETURNS users AS $$
DECLARE
    v_user users%ROWTYPE;
BEGIN
    SELECT * INTO v_user FROM users WHERE id = p_user_id;
    RETURN v_user;
END;
$$ LANGUAGE plpgsql;
```

Prefix parameters with `p_` and variables with `v_` to avoid ambiguity with column names.

### Use STRICT for Single-Row Queries

```sql
-- Without STRICT: silently uses first row
SELECT name INTO user_name FROM users WHERE status = 'active';

-- With STRICT: raises error if not exactly 1 row
SELECT name INTO STRICT user_name FROM users WHERE id = p_id;
```

### Prefer SQL Over Loops

```sql
-- BAD: Row-by-row processing
FOR rec IN SELECT * FROM orders WHERE status = 'pending' LOOP
    UPDATE orders SET status = 'processing' WHERE id = rec.id;
END LOOP;

-- GOOD: Single SQL statement
UPDATE orders SET status = 'processing' WHERE status = 'pending';
```

### Function Volatility

```sql
-- IMMUTABLE: Same inputs always return same output
CREATE FUNCTION calculate_tax(amount NUMERIC)
RETURNS NUMERIC AS $$
BEGIN RETURN amount * 0.08; END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- STABLE: Same result within single query
CREATE FUNCTION get_current_rate()
RETURNS NUMERIC AS $$
BEGIN
    RETURN (SELECT rate FROM config WHERE key = 'tax_rate');
END;
$$ LANGUAGE plpgsql STABLE;

-- VOLATILE: May return different results each call (default)
CREATE FUNCTION log_access()
RETURNS VOID AS $$
BEGIN INSERT INTO access_log (time) VALUES (NOW()); END;
$$ LANGUAGE plpgsql VOLATILE;
```

### Security Best Practices

```sql
-- SECURITY DEFINER runs as function owner
-- Always set search_path to prevent hijacking
CREATE FUNCTION safe_function()
RETURNS VOID AS $$
BEGIN
    -- operations
END;
$$ LANGUAGE plpgsql
   SECURITY DEFINER
   SET search_path = public, pg_temp;
```

### Minimize Context Switches

```sql
-- BAD: Multiple queries for the same row
SELECT name INTO v_name FROM users WHERE id = p_id;
SELECT email INTO v_email FROM users WHERE id = p_id;

-- GOOD: Single query
SELECT name, email INTO v_name, v_email FROM users WHERE id = p_id;
```

## Common Pitfalls

1. **Marking a VOLATILE function as IMMUTABLE** -- This can cause incorrect cached results and corrupt index data. Only use IMMUTABLE when the function truly has no side effects and always returns the same result for the same inputs.
2. **SECURITY DEFINER without SET search_path** -- An attacker can create objects in a schema that appears earlier in the search path, hijacking your function's queries.

## Best Practices

1. **Set the correct volatility category** -- IMMUTABLE for pure calculations, STABLE for read-only functions within a transaction, VOLATILE for anything with side effects.
2. **Always pair SECURITY DEFINER with SET search_path** -- This prevents search path manipulation attacks.

## Summary

- Naming conventions (p_ for parameters, v_ for variables) prevent column-name collisions and improve readability.
- Function volatility (IMMUTABLE, STABLE, VOLATILE) controls query planner optimizations and index eligibility.
- SECURITY DEFINER must always be paired with SET search_path to prevent security vulnerabilities.

## Code Examples

**Demonstrates IMMUTABLE volatility for index-eligible functions and the critical SECURITY DEFINER + SET search_path pairing.**

```sql
-- IMMUTABLE: usable in indexes, cached by planner
CREATE FUNCTION calculate_tax(amount NUMERIC)
RETURNS NUMERIC AS $$
BEGIN RETURN amount * 0.08; END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- SECURITY DEFINER: always set search_path
CREATE FUNCTION admin_operation()
RETURNS VOID AS $$
BEGIN /* ... */ END;
$$ LANGUAGE plpgsql SECURITY DEFINER SET search_path = public, pg_temp;
```


## Resources

- [Development Tips](https://www.postgresql.org/docs/18/plpgsql-development-tips.html) — Tips for PL/pgSQL development

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*