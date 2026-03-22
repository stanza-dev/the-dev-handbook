---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-variables"
---

# Variables and Data Types

## Introduction

Variables are the building blocks of any procedural program. In PL/pgSQL, variables store intermediate results, loop counters, and fetched rows while your function executes. Understanding variable declarations, type inference with `%TYPE` and `%ROWTYPE`, and composite types will let you write robust, type-safe server-side code.

## Key Concepts

- **%TYPE**: A type-copying mechanism that makes a variable inherit its data type from a table column or another variable.
- **%ROWTYPE**: Declares a variable that can hold an entire row from a specified table, with one field per column.
- **RECORD**: A generic row type whose structure is determined at runtime by whatever query result is assigned to it.
- **CONSTANT**: A variable modifier that prevents reassignment after initialization.

## Real World Context

Consider a billing function that reads an invoice row, applies discounts, computes tax, and writes the result. Using `%ROWTYPE` means your function automatically adapts when the invoice table gains new columns, and using `%TYPE` for the total ensures the variable matches the column precision exactly -- no silent truncation bugs.

## Deep Dive

### Declaring Variables

```sql
DECLARE
    user_id INTEGER;
    username VARCHAR(100);
    price NUMERIC(10, 2);
    is_active BOOLEAN;
    created_at TIMESTAMP;

    -- With default values
    counter INTEGER := 0;
    status TEXT DEFAULT 'pending';

    -- Constants
    TAX_RATE CONSTANT NUMERIC := 0.08;

    -- NOT NULL constraint
    total NUMERIC NOT NULL := 0;
```

### Type Inference with %TYPE

Copy a type from a column or variable so your code adapts automatically if the column type changes:

```sql
DECLARE
    user_email users.email%TYPE;
    email_copy user_email%TYPE;
```

### Row Types with %ROWTYPE

Declare a variable to hold an entire row:

```sql
DECLARE
    user_record users%ROWTYPE;
BEGIN
    SELECT * INTO user_record FROM users WHERE id = 1;
    RAISE NOTICE 'User: % (%)', user_record.name, user_record.email;
END;
```

### RECORD Type

A generic record that adapts to any row structure:

```sql
DECLARE
    rec RECORD;
BEGIN
    SELECT id, name INTO rec FROM users WHERE id = 1;
    RAISE NOTICE 'ID: %, Name: %', rec.id, rec.name;
END;
```

### Array Variables

PostgreSQL 18 adds `array_sort()` and `array_reverse()` as built-in functions, making array manipulation easier:

```sql
DECLARE
    tags TEXT[];
    numbers INTEGER[] := ARRAY[5, 3, 1, 4, 2];
BEGIN
    tags := tags || 'new_tag';
    RAISE NOTICE 'Sorted: %', array_sort(numbers);      -- {1,2,3,4,5}
    RAISE NOTICE 'Reversed: %', array_reverse(numbers);  -- {2,4,1,3,5}
    RAISE NOTICE 'Element: %', numbers[1];  -- 1-indexed!
END;
```

### Composite Types

```sql
CREATE TYPE address AS (
    street TEXT,
    city TEXT,
    zip_code VARCHAR(10)
);

CREATE FUNCTION get_address(user_id INT) RETURNS address AS $$
DECLARE
    result address;
BEGIN
    SELECT street, city, zip_code
    INTO result.street, result.city, result.zip_code
    FROM addresses
    WHERE user_id = $1;
    RETURN result;
END;
$$ LANGUAGE plpgsql;
```

## Common Pitfalls

1. **Forgetting that arrays are 1-indexed** -- Unlike most programming languages, PostgreSQL arrays start at index 1. Accessing `arr[0]` returns NULL, not the first element.
2. **Using RECORD without assigning first** -- A RECORD variable has no structure until a query result is assigned. Accessing fields before assignment raises an error.

## Best Practices

1. **Use %TYPE for column-derived variables** -- This ensures your function stays in sync with schema changes without manual updates.
2. **Prefer %ROWTYPE over RECORD when the table is known** -- %ROWTYPE gives you compile-time checking, while RECORD defers all checks to runtime.

## Summary

- PL/pgSQL variables support all PostgreSQL data types, constants, NOT NULL constraints, and default values.
- `%TYPE` and `%ROWTYPE` provide type inference that automatically adapts to schema changes.
- PostgreSQL 18 introduces `array_sort()` and `array_reverse()` for convenient in-place array manipulation.

## Code Examples

**Shows %TYPE, %ROWTYPE, RECORD, CONSTANT declarations, and the new PostgreSQL 18 array_sort() function.**

```sql
DECLARE
    user_email users.email%TYPE;          -- copies column type
    user_record users%ROWTYPE;            -- holds entire row
    rec RECORD;                           -- generic row
    TAX_RATE CONSTANT NUMERIC := 0.08;    -- immutable
    numbers INTEGER[] := ARRAY[5,3,1,4,2];
BEGIN
    SELECT * INTO user_record FROM users WHERE id = 1;
    RAISE NOTICE 'User: %', user_record.name;
    RAISE NOTICE 'Sorted: %', array_sort(numbers);  -- PG18
END;
```


## Resources

- [PL/pgSQL Declarations](https://www.postgresql.org/docs/18/plpgsql-declarations.html) — Variable declarations

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*