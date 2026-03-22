---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-conditionals"
---

# Conditional Statements

## Introduction

Conditional statements let your PL/pgSQL code make decisions at runtime. Whether you need a simple yes/no branch or a multi-way dispatch, IF and CASE statements are the tools you will use every day. Understanding their subtle differences -- especially around NULL handling and missing ELSE clauses -- will save you from hard-to-debug production issues.

## Key Concepts

- **IF/ELSIF/ELSE**: The primary branching construct. Evaluates conditions sequentially and executes the first matching branch.
- **Simple CASE**: Compares a single expression against a list of values, similar to a switch statement in other languages.
- **Searched CASE**: Evaluates independent boolean conditions, offering more flexibility than simple CASE.
- **CASE_NOT_FOUND**: An exception raised when a CASE statement has no ELSE clause and no condition matches.

## Real World Context

A shipping cost calculator for an e-commerce platform must apply different rates based on order weight, destination zone, and membership tier. IF/ELSIF chains handle the tiered logic, while CASE statements cleanly dispatch on enumerated zone codes. Getting the NULL handling right ensures that orders with missing weight data fail gracefully instead of silently charging zero.

## Deep Dive

### IF-THEN-ELSE

```sql
-- Simple IF
IF condition THEN
    statements;
END IF;

-- IF-ELSE
IF condition THEN
    statements;
ELSE
    other_statements;
END IF;

-- IF-ELSIF-ELSE
IF condition1 THEN
    statements1;
ELSIF condition2 THEN
    statements2;
ELSE
    default_statements;
END IF;
```

### Practical Example

```sql
CREATE FUNCTION get_order_status(order_total NUMERIC)
RETURNS TEXT AS $$
BEGIN
    IF order_total IS NULL THEN
        RETURN 'Invalid order';
    ELSIF order_total < 0 THEN
        RETURN 'Error: negative total';
    ELSIF order_total = 0 THEN
        RETURN 'Empty order';
    ELSIF order_total < 50 THEN
        RETURN 'Small order';
    ELSIF order_total < 200 THEN
        RETURN 'Medium order';
    ELSE
        RETURN 'Large order';
    END IF;
END;
$$ LANGUAGE plpgsql;
```

### Simple CASE

```sql
CREATE FUNCTION day_type(day_num INTEGER) RETURNS TEXT AS $$
BEGIN
    CASE day_num
        WHEN 1, 7 THEN
            RETURN 'Weekend';
        WHEN 2, 3, 4, 5, 6 THEN
            RETURN 'Weekday';
        ELSE
            RETURN 'Invalid day';
    END CASE;
END;
$$ LANGUAGE plpgsql;
```

### Searched CASE

```sql
CREATE FUNCTION categorize_age(age INTEGER) RETURNS TEXT AS $$
BEGIN
    CASE
        WHEN age < 0 THEN RETURN 'Invalid';
        WHEN age < 13 THEN RETURN 'Child';
        WHEN age < 20 THEN RETURN 'Teenager';
        WHEN age < 65 THEN RETURN 'Adult';
        ELSE RETURN 'Senior';
    END CASE;
END;
$$ LANGUAGE plpgsql;
```

### CASE vs IF

CASE raises an exception if no condition matches and there is no ELSE. IF simply does nothing:

```sql
-- This raises CASE_NOT_FOUND if x > 20
CASE
    WHEN x BETWEEN 0 AND 10 THEN msg := 'low';
    WHEN x BETWEEN 11 AND 20 THEN msg := 'medium';
END CASE;
```

### NULL Handling in Conditions

```sql
IF value = NULL THEN  -- Always FALSE! NULL is not equal to anything.
END IF;

IF value IS NULL THEN  -- Correct
END IF;

IF COALESCE(value, 0) > 100 THEN  -- Safe default
END IF;
```

## Common Pitfalls

1. **Comparing with `= NULL`** -- This is always false. Use `IS NULL` or `IS NOT NULL` instead.
2. **Omitting ELSE in CASE** -- Unlike IF (which silently does nothing), a CASE without ELSE raises `CASE_NOT_FOUND` if no branch matches.

## Best Practices

1. **Always include ELSE in CASE statements** -- Even if you just re-raise or assign a default, it prevents unexpected exceptions.
2. **Handle NULL explicitly at the top of conditionals** -- Check for NULL before doing arithmetic or string comparisons to avoid surprises.

## Summary

- IF/ELSIF/ELSE is the primary branching construct; it silently falls through if no branch matches.
- CASE statements come in simple (value-matching) and searched (condition-checking) forms.
- CASE raises CASE_NOT_FOUND when no ELSE is provided and no branch matches, unlike IF which does nothing.

## Code Examples

**An IF/ELSIF/ELSE chain that classifies orders by total, with explicit NULL handling as the first check.**

```sql
CREATE FUNCTION get_order_status(order_total NUMERIC)
RETURNS TEXT AS $$
BEGIN
    IF order_total IS NULL THEN
        RETURN 'Invalid order';
    ELSIF order_total < 50 THEN
        RETURN 'Small order';
    ELSIF order_total < 200 THEN
        RETURN 'Medium order';
    ELSE
        RETURN 'Large order';
    END IF;
END;
$$ LANGUAGE plpgsql;
```


## Resources

- [Control Structures](https://www.postgresql.org/docs/18/plpgsql-control-structures.html) — Complete control structures documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*