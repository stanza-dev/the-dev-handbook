---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-block-structure"
---

# PL/pgSQL Block Structure

All PL/pgSQL code is organized in blocks with a specific structure.

## Basic Block Structure

```plpgsql
[ <<label>> ]
[ DECLARE
    -- Variable declarations ]
BEGIN
    -- Statements
[ EXCEPTION
    -- Error handlers ]
END [ label ];
```

## Simple Example

```plpgsql
DO $$
DECLARE
    message TEXT := 'Hello, PL/pgSQL!';
BEGIN
    RAISE NOTICE '%', message;
END;
$$;
```

The `DO` command executes an anonymous block (no need to create a function).

## Complete Function Example

```plpgsql
CREATE OR REPLACE FUNCTION calculate_discount(
    original_price NUMERIC,
    discount_percent NUMERIC
) RETURNS NUMERIC AS $$
DECLARE
    discount_amount NUMERIC;
    final_price NUMERIC;
BEGIN
    -- Calculate the discount
    discount_amount := original_price * (discount_percent / 100);
    final_price := original_price - discount_amount;
    
    -- Ensure price is not negative
    IF final_price < 0 THEN
        final_price := 0;
    END IF;
    
    RETURN final_price;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT calculate_discount(100, 15);  -- Returns 85.00
```

## Nested Blocks

Blocks can be nested for scope control:

```plpgsql
CREATE FUNCTION nested_example() RETURNS TEXT AS $$
DECLARE
    outer_var TEXT := 'outer';
BEGIN
    <<inner_block>>
    DECLARE
        inner_var TEXT := 'inner';
    BEGIN
        -- Can access both outer_var and inner_var
        RETURN outer_var || ' and ' || inner_var;
    END inner_block;
END;
$$ LANGUAGE plpgsql;
```

## Dollar Quoting

Use `$$` or named tags to avoid escaping quotes:

```plpgsql
-- Using $$ (most common)
CREATE FUNCTION greeting() RETURNS TEXT AS $$
BEGIN
    RETURN 'Hello, World!';
END;
$$ LANGUAGE plpgsql;

-- Using named tags (for nested functions)
CREATE FUNCTION outer_func() RETURNS TEXT AS $outer$
DECLARE
    inner_func TEXT := $inner$
        SELECT 'nested string with $ signs'
    $inner$;
BEGIN
    RETURN inner_func;
END;
$outer$ LANGUAGE plpgsql;
```

## Important Notes

1. **Semicolons required** after every statement and declaration
2. **Assignment uses `:=`** (not `=`)
3. **LANGUAGE clause is required** when creating functions
4. **OR REPLACE** allows modifying existing functions

📖 [Block Structure](https://www.postgresql.org/docs/18/plpgsql-structure.html)

## Resources

- [Structure of PL/pgSQL](https://www.postgresql.org/docs/18/plpgsql-structure.html) — Block structure details

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*