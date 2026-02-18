---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-variables"
---

# Variables and Data Types

Variables store intermediate results and parameters in PL/pgSQL.

## Declaring Variables

```plpgsql
DECLARE
    -- Basic types
    user_id INTEGER;
    username VARCHAR(100);
    email TEXT;
    price NUMERIC(10, 2);
    is_active BOOLEAN;
    created_at TIMESTAMP;
    
    -- With default values
    counter INTEGER := 0;
    status TEXT DEFAULT 'pending';
    
    -- Constants (cannot be changed)
    TAX_RATE CONSTANT NUMERIC := 0.08;
    
    -- NOT NULL (must have default or be assigned before use)
    total NUMERIC NOT NULL := 0;
```

## Type Inference with %TYPE

Copy a type from a column or variable:

```plpgsql
DECLARE
    -- Same type as users.email column
    user_email users.email%TYPE;
    
    -- Same type as another variable
    email_copy user_email%TYPE;
```

**Benefit**: If the column type changes, your code adapts automatically.

## Row Types with %ROWTYPE

Declare a variable to hold an entire row:

```plpgsql
DECLARE
    -- Can hold any row from users table
    user_record users%ROWTYPE;
BEGIN
    SELECT * INTO user_record FROM users WHERE id = 1;
    
    -- Access fields with dot notation
    RAISE NOTICE 'User: % (%)', user_record.name, user_record.email;
END;
```

## RECORD Type

Generic record that can hold any row structure:

```plpgsql
DECLARE
    rec RECORD;
BEGIN
    -- Can hold result of any query
    SELECT id, name INTO rec FROM users WHERE id = 1;
    RAISE NOTICE 'ID: %, Name: %', rec.id, rec.name;
    
    -- Or a different query structure
    SELECT COUNT(*) AS cnt INTO rec FROM orders;
    RAISE NOTICE 'Count: %', rec.cnt;
END;
```

## Array Variables

```plpgsql
DECLARE
    tags TEXT[];
    numbers INTEGER[] := ARRAY[1, 2, 3, 4, 5];
    matrix INTEGER[][] := '{{1,2},{3,4}}';
BEGIN
    -- Append to array
    tags := tags || 'new_tag';
    
    -- Access elements (1-indexed!)
    RAISE NOTICE 'First: %', numbers[1];
    
    -- Slice
    RAISE NOTICE 'Slice: %', numbers[2:4];
END;
```

## Composite Types

```plpgsql
-- Create a custom type
CREATE TYPE address AS (
    street TEXT,
    city TEXT,
    zip_code VARCHAR(10)
);

-- Use in function
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

📖 [Declarations](https://www.postgresql.org/docs/18/plpgsql-declarations.html)

## Resources

- [PL/pgSQL Declarations](https://www.postgresql.org/docs/18/plpgsql-declarations.html) — Variable declarations

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*