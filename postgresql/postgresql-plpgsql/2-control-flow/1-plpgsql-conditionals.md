---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-conditionals"
---

# Conditional Statements

Control program flow based on conditions.

## IF-THEN-ELSE

```plpgsql
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
ELSIF condition3 THEN
    statements3;
ELSE
    default_statements;
END IF;
```

## Practical Example

```plpgsql
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

## CASE Statements

### Simple CASE

```plpgsql
CASE search_expression
    WHEN expression1 THEN
        statements1;
    WHEN expression2 THEN
        statements2;
    ELSE
        default_statements;
END CASE;
```

Example:
```plpgsql
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

```plpgsql
CREATE FUNCTION categorize_age(age INTEGER) RETURNS TEXT AS $$
BEGIN
    CASE
        WHEN age < 0 THEN
            RETURN 'Invalid';
        WHEN age < 13 THEN
            RETURN 'Child';
        WHEN age < 20 THEN
            RETURN 'Teenager';
        WHEN age < 65 THEN
            RETURN 'Adult';
        ELSE
            RETURN 'Senior';
    END CASE;
END;
$$ LANGUAGE plpgsql;
```

## CASE vs IF

- **CASE** raises exception if no condition matches and no ELSE
- **IF** does nothing if no condition matches and no ELSE

```plpgsql
-- This will raise CASE_NOT_FOUND exception if x > 20
CASE
    WHEN x BETWEEN 0 AND 10 THEN
        msg := 'low';
    WHEN x BETWEEN 11 AND 20 THEN
        msg := 'medium';
    -- No ELSE!
END CASE;
```

## Null Handling in Conditions

```plpgsql
-- NULL comparisons are tricky!
IF value = NULL THEN  -- Always false!
    -- This never executes
END IF;

-- Correct way:
IF value IS NULL THEN
    -- Handle NULL
END IF;

-- Using COALESCE
IF COALESCE(value, 0) > 100 THEN
    -- Handle case
END IF;
```

📖 [Control Structures](https://www.postgresql.org/docs/18/plpgsql-control-structures.html)

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*