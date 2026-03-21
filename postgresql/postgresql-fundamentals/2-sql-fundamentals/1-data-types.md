---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-data-types"
---

# PostgreSQL Data Types

## Introduction
PostgreSQL offers one of the richest sets of native data types of any database. Choosing the right type improves performance, saves storage, and prevents data errors. This lesson covers the types you will use most often.

## Key Concepts
- **NUMERIC(p,s)**: Exact-precision decimal type with p total digits and s digits after the decimal point. Essential for monetary values.
- **VARCHAR(n) vs TEXT**: Both store variable-length text. Use VARCHAR(n) when you need to enforce a maximum length; use TEXT when you don't.
- **TIMESTAMPTZ**: A timestamp that stores timezone information. Always prefer this over plain TIMESTAMP for real applications.
- **UUID**: A 128-bit universally unique identifier, useful for distributed systems and public-facing IDs.

## Real World Context
Choosing `REAL` instead of `NUMERIC` for prices can cause rounding errors that cost real money. Using `TIMESTAMP` instead of `TIMESTAMPTZ` leads to timezone bugs when your users span multiple time zones. Getting data types right from the start prevents painful migrations later.

## Deep Dive

### Numeric Types

PostgreSQL offers several numeric types for different use cases:

| Type | Storage | Description | Range |
|------|---------|-------------|-------|
| `SMALLINT` | 2 bytes | Small integer | -32,768 to 32,767 |
| `INTEGER` | 4 bytes | Typical integer | -2 billion to 2 billion |
| `BIGINT` | 8 bytes | Large integer | -9 quintillion to 9 quintillion |
| `NUMERIC(p,s)` | Variable | Exact precision | Up to 131,072 digits |
| `REAL` | 4 bytes | Floating point | 6 decimal digits precision |
| `DOUBLE PRECISION` | 8 bytes | Floating point | 15 decimal digits precision |

For monetary values, always use NUMERIC to avoid floating-point rounding errors:

```sql
-- Use NUMERIC for money (exact precision)
CREATE TABLE products (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100),
    price NUMERIC(10, 2),  -- 10 digits total, 2 after decimal
    quantity INTEGER
);
```

The `NUMERIC(10, 2)` type ensures prices like `1234.56` are stored exactly, with no rounding surprises.

### Character Types

| Type | Description |
|------|-------------|
| `VARCHAR(n)` | Variable-length with limit |
| `CHAR(n)` | Fixed-length, blank-padded |
| `TEXT` | Variable unlimited length |

In PostgreSQL, `TEXT` performs identically to `VARCHAR` — there is no performance penalty for using TEXT:

```sql
-- TEXT is often preferred - no performance penalty
CREATE TABLE articles (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title VARCHAR(255),    -- Good for constrained lengths
    content TEXT,          -- Unlimited length
    slug VARCHAR(100) UNIQUE
);
```

Use `VARCHAR(n)` only when you need to enforce a maximum length as a business rule.

### Date/Time Types

| Type | Description | Example |
|------|-------------|--------|
| `DATE` | Date only | '2024-01-15' |
| `TIME` | Time only | '14:30:00' |
| `TIMESTAMP` | Date and time | '2024-01-15 14:30:00' |
| `TIMESTAMPTZ` | With timezone | '2024-01-15 14:30:00+00' |
| `INTERVAL` | Time span | '1 day 2 hours' |

Always use TIMESTAMPTZ for columns that store points in time:

```sql
CREATE TABLE events (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100),
    starts_at TIMESTAMPTZ NOT NULL,
    ends_at TIMESTAMPTZ,
    duration INTERVAL
);
```

TIMESTAMPTZ converts to UTC for storage and converts back to the client's timezone on retrieval, preventing timezone bugs.

### Boolean Type

PostgreSQL has a native boolean type that accepts multiple input formats:

```sql
CREATE TABLE users (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    is_active BOOLEAN DEFAULT true,
    is_admin BOOLEAN DEFAULT false
);
```

Valid boolean values include: `TRUE`, `FALSE`, `'t'`, `'f'`, `'yes'`, `'no'`, `'on'`, `'off'`, `1`, `0`.

### UUID Type

PostgreSQL 13+ has a built-in `gen_random_uuid()` function — no extension needed:

```sql
-- PostgreSQL 13+: built-in, no extension needed
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id INTEGER REFERENCES users(id),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- PostgreSQL 18: also supports uuidv7() (time-ordered)
CREATE TABLE events_log (
    id UUID PRIMARY KEY DEFAULT uuidv7(),  -- Time-ordered UUID
    event_type VARCHAR(50),
    payload JSONB
);
```

The `uuidv7()` function in PostgreSQL 18 generates time-ordered UUIDs, which are more efficient for indexing because they are roughly sequential.

## Common Pitfalls
1. **Using REAL or DOUBLE PRECISION for money** — Floating-point types introduce rounding errors. Use `NUMERIC(p,s)` for any value that requires exact decimal precision.
2. **Using TIMESTAMP instead of TIMESTAMPTZ** — Without timezone information, timestamps become ambiguous when your application serves users in different time zones. Always use TIMESTAMPTZ.
3. **Using uuid-ossp extension for UUID generation** — The `uuid-ossp` extension is no longer needed in PostgreSQL 13+. Use the built-in `gen_random_uuid()` instead.

## Best Practices
1. **Default to TEXT for strings** — Unless you need a length constraint as a business rule, TEXT is simpler and performs identically to VARCHAR.
2. **Use TIMESTAMPTZ for all timestamps** — It stores timezone information and handles conversions automatically.
3. **Use gen_random_uuid() or uuidv7()** — Built-in functions are simpler and don't require installing extensions.

## Summary
- Use NUMERIC for exact decimal values (money, measurements) and INTEGER/BIGINT for whole numbers.
- TEXT and VARCHAR perform the same in PostgreSQL; use VARCHAR(n) only when enforcing a max length.
- Always use TIMESTAMPTZ instead of TIMESTAMP for storing points in time.
- PostgreSQL 13+ has built-in UUID generation; PostgreSQL 18 adds uuidv7() for time-ordered UUIDs.
- Choose data types carefully at the start to avoid painful migrations later.

## Code Examples

**Comparing NUMERIC vs REAL precision and generating UUIDs with the built-in function**

```sql
-- Choosing the right numeric type
-- Exact precision for money:
SELECT 0.1::NUMERIC + 0.2::NUMERIC;  -- 0.3 (exact)
SELECT 0.1::REAL + 0.2::REAL;        -- 0.30000001 (imprecise!)

-- UUID generation (no extension needed in PG 13+)
SELECT gen_random_uuid();
-- Output: 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11'
```


## Resources

- [PostgreSQL Data Types](https://www.postgresql.org/docs/18/datatype.html) — Complete reference for all PostgreSQL data types

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*