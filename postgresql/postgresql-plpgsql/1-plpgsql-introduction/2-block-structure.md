---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-block-structure"
---

# PL/pgSQL Block Structure

## Introduction

Every piece of PL/pgSQL code lives inside a block. Understanding blocks is essential because they define where you declare variables, where you write logic, and how you handle errors. Mastering block structure is the foundation for writing any PL/pgSQL function or procedure.

## Key Concepts

- **Block**: The fundamental unit of PL/pgSQL code, consisting of an optional DECLARE section, a mandatory BEGIN/END section, and an optional EXCEPTION section.
- **Dollar Quoting**: A PostgreSQL string-quoting mechanism (`$$` or `$tag$`) that lets you write PL/pgSQL bodies without escaping single quotes.
- **Nested Block**: A block inside another block, used for scope control and localized exception handling.
- **Label**: An optional `<<name>>` tag before a block that enables referencing it from EXIT or CONTINUE statements.

## Real World Context

When you write a migration script that processes thousands of rows, you often need temporary variables, localized error handling, and clear scope boundaries. Block structure gives you all of this. A well-structured function with nested blocks makes it easy for teammates to understand which variables are in scope and where errors are caught.

## Deep Dive

### Basic Block Structure

```sql
[ <<label>> ]
[ DECLARE
    -- Variable declarations ]
BEGIN
    -- Statements
[ EXCEPTION
    -- Error handlers ]
END [ label ];
```

### Simple Example

The `DO` command executes an anonymous block without creating a persistent function:

```sql
DO $$
DECLARE
    message TEXT := 'Hello, PL/pgSQL!';
BEGIN
    RAISE NOTICE '%', message;
END;
$$;
```

### Complete Function Example

Here is a function that uses DECLARE for local variables and IF for control flow:

```sql
CREATE OR REPLACE FUNCTION calculate_discount(
    original_price NUMERIC,
    discount_percent NUMERIC
) RETURNS NUMERIC AS $$
DECLARE
    discount_amount NUMERIC;
    final_price NUMERIC;
BEGIN
    discount_amount := original_price * (discount_percent / 100);
    final_price := original_price - discount_amount;

    IF final_price < 0 THEN
        final_price := 0;
    END IF;

    RETURN final_price;
END;
$$ LANGUAGE plpgsql;

SELECT calculate_discount(100, 15);  -- Returns 85.00
```

### Nested Blocks

Blocks can be nested for scope control. Inner blocks can see outer variables, but not vice versa:

```sql
CREATE FUNCTION nested_example() RETURNS TEXT AS $$
DECLARE
    outer_var TEXT := 'outer';
BEGIN
    <<inner_block>>
    DECLARE
        inner_var TEXT := 'inner';
    BEGIN
        RETURN outer_var || ' and ' || inner_var;
    END inner_block;
END;
$$ LANGUAGE plpgsql;
```

### Dollar Quoting

Use `$$` or named tags to avoid escaping single quotes inside your function body:

```sql
-- Using named tags for nested dollar-quoted strings
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

## Common Pitfalls

1. **Forgetting semicolons** -- Every statement and declaration in PL/pgSQL must end with a semicolon. Missing one produces a cryptic parse error at the `END` keyword.
2. **Using `=` instead of `:=` for assignment** -- In PL/pgSQL, `=` is only valid in SQL expressions and comparisons. Variable assignment requires `:=`.

## Best Practices

1. **Always use OR REPLACE** -- `CREATE OR REPLACE FUNCTION` lets you iterate during development without dropping and recreating the function each time.
2. **Use named dollar-quote tags in nested contexts** -- When a function body itself contains dollar-quoted strings, use `$outer$` / `$inner$` tags to avoid ambiguity.

## Summary

- PL/pgSQL code is organized into blocks with DECLARE, BEGIN, EXCEPTION, and END sections.
- Blocks can be nested for scope isolation, and labels let you reference specific blocks from EXIT or CONTINUE.
- Dollar quoting (`$$`) avoids the need to escape single quotes inside function bodies.

## Code Examples

**A complete PL/pgSQL function showing DECLARE, BEGIN/END block structure, variable assignment with :=, and a simple IF conditional.**

```sql
CREATE OR REPLACE FUNCTION calculate_discount(
    original_price NUMERIC,
    discount_percent NUMERIC
) RETURNS NUMERIC AS $$
DECLARE
    discount_amount NUMERIC;
    final_price NUMERIC;
BEGIN
    discount_amount := original_price * (discount_percent / 100);
    final_price := original_price - discount_amount;

    IF final_price < 0 THEN
        final_price := 0;
    END IF;

    RETURN final_price;
END;
$$ LANGUAGE plpgsql;
```


## Resources

- [Structure of PL/pgSQL](https://www.postgresql.org/docs/18/plpgsql-structure.html) — Block structure details

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*