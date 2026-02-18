---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-loops"
---

# Loop Structures

PL/pgSQL provides several loop types for iteration.

## Basic LOOP

Infinite loop - must use EXIT:

```plpgsql
LOOP
    -- statements
    EXIT WHEN condition;  -- or just EXIT to exit unconditionally
END LOOP;
```

Example:
```plpgsql
CREATE FUNCTION countdown(start_num INTEGER) RETURNS TEXT AS $$
DECLARE
    counter INTEGER := start_num;
    result TEXT := '';
BEGIN
    LOOP
        result := result || counter::TEXT || ' ';
        counter := counter - 1;
        EXIT WHEN counter < 0;
    END LOOP;
    RETURN result;
END;
$$ LANGUAGE plpgsql;
-- countdown(5) returns '5 4 3 2 1 0 '
```

## WHILE Loop

```plpgsql
WHILE condition LOOP
    statements;
END LOOP;
```

Example:
```plpgsql
CREATE FUNCTION factorial(n INTEGER) RETURNS BIGINT AS $$
DECLARE
    result BIGINT := 1;
    counter INTEGER := n;
BEGIN
    WHILE counter > 1 LOOP
        result := result * counter;
        counter := counter - 1;
    END LOOP;
    RETURN result;
END;
$$ LANGUAGE plpgsql;
```

## FOR Loop (Integer Range)

```plpgsql
-- Ascending
FOR i IN 1..10 LOOP
    -- i takes values 1, 2, 3, ..., 10
END LOOP;

-- Descending
FOR i IN REVERSE 10..1 LOOP
    -- i takes values 10, 9, 8, ..., 1
END LOOP;

-- With step
FOR i IN 1..10 BY 2 LOOP
    -- i takes values 1, 3, 5, 7, 9
END LOOP;
```

Example:
```plpgsql
CREATE FUNCTION sum_range(start_val INT, end_val INT) RETURNS INT AS $$
DECLARE
    total INT := 0;
BEGIN
    FOR i IN start_val..end_val LOOP
        total := total + i;
    END LOOP;
    RETURN total;
END;
$$ LANGUAGE plpgsql;
```

## FOR Loop (Query Results)

```plpgsql
CREATE FUNCTION process_users() RETURNS VOID AS $$
DECLARE
    user_rec RECORD;
BEGIN
    FOR user_rec IN SELECT id, name, email FROM users WHERE active = true LOOP
        RAISE NOTICE 'Processing user: % (%)', user_rec.name, user_rec.email;
        -- Process each user
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

## FOREACH (Arrays)

```plpgsql
CREATE FUNCTION sum_array(arr INTEGER[]) RETURNS INTEGER AS $$
DECLARE
    element INTEGER;
    total INTEGER := 0;
BEGIN
    FOREACH element IN ARRAY arr LOOP
        total := total + element;
    END LOOP;
    RETURN total;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT sum_array(ARRAY[1, 2, 3, 4, 5]);  -- Returns 15
```

## Loop Control

```plpgsql
-- EXIT: leave loop
EXIT;                    -- Exit immediately
EXIT WHEN condition;     -- Exit if condition is true
EXIT label;              -- Exit named loop

-- CONTINUE: skip to next iteration
CONTINUE;                -- Skip rest of current iteration
CONTINUE WHEN condition; -- Skip if condition is true
```

Labeled loops:
```plpgsql
<<outer_loop>>
FOR i IN 1..10 LOOP
    <<inner_loop>>
    FOR j IN 1..10 LOOP
        IF some_condition THEN
            EXIT outer_loop;  -- Exit the outer loop
        END IF;
    END LOOP inner_loop;
END LOOP outer_loop;
```

📖 [Control Structures](https://www.postgresql.org/docs/18/plpgsql-control-structures.html)

## Resources

- [Loop Statements](https://www.postgresql.org/docs/18/plpgsql-control-structures.html#PLPGSQL-CONTROL-STRUCTURES-LOOPS) — Complete loop documentation

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*