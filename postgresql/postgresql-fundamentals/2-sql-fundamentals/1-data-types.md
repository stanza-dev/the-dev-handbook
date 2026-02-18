---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-data-types"
---

# PostgreSQL Data Types

PostgreSQL offers a rich set of native data types. Choosing the right type improves performance, saves storage, and prevents data errors.

## Numeric Types

| Type | Storage | Description | Range |
|------|---------|-------------|-------|
| `SMALLINT` | 2 bytes | Small integer | -32,768 to 32,767 |
| `INTEGER` | 4 bytes | Typical integer | -2 billion to 2 billion |
| `BIGINT` | 8 bytes | Large integer | -9 quintillion to 9 quintillion |
| `NUMERIC(p,s)` | Variable | Exact precision | Up to 131,072 digits |
| `REAL` | 4 bytes | Floating point | 6 decimal digits precision |
| `DOUBLE PRECISION` | 8 bytes | Floating point | 15 decimal digits precision |

```sql
-- Use NUMERIC for money (exact precision)
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price NUMERIC(10, 2),  -- 10 digits total, 2 after decimal
    quantity INTEGER
);
```

## Character Types

| Type | Description |
|------|-------------|
| `VARCHAR(n)` | Variable-length with limit |
| `CHAR(n)` | Fixed-length, blank-padded |
| `TEXT` | Variable unlimited length |

```sql
-- TEXT is often preferred - no performance penalty
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),    -- Good for constrained lengths
    content TEXT,          -- Unlimited length
    slug VARCHAR(100) UNIQUE
);
```

**Tip**: In PostgreSQL, `TEXT` performs identically to `VARCHAR`. Use `VARCHAR(n)` only when you need to enforce a length limit.

## Date/Time Types

| Type | Description | Example |
|------|-------------|--------|
| `DATE` | Date only | '2024-01-15' |
| `TIME` | Time only | '14:30:00' |
| `TIMESTAMP` | Date and time | '2024-01-15 14:30:00' |
| `TIMESTAMPTZ` | With timezone | '2024-01-15 14:30:00+00' |
| `INTERVAL` | Time span | '1 day 2 hours' |

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    starts_at TIMESTAMPTZ NOT NULL,
    ends_at TIMESTAMPTZ,
    duration INTERVAL
);

-- Always use TIMESTAMPTZ for real applications!
```

## Boolean Type

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    is_active BOOLEAN DEFAULT true,
    is_admin BOOLEAN DEFAULT false
);
```

Valid boolean values: `TRUE`, `FALSE`, `'t'`, `'f'`, `'yes'`, `'no'`, `'on'`, `'off'`, `1`, `0`

## UUID Type

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id INTEGER REFERENCES users(id),
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

📖 [Data Types Documentation](https://www.postgresql.org/docs/18/datatype.html)

## Resources

- [PostgreSQL Data Types](https://www.postgresql.org/docs/18/datatype.html) — Complete reference for all PostgreSQL data types

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*