---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-row-column-filtering"
---

# Row and Column Filtering

## Introduction

PostgreSQL 15+ supports filtering which rows and columns are replicated through logical replication. This enables selective data distribution, privacy compliance, and efficient bandwidth usage.

This lesson covers row filtering with WHERE clauses, column lists for selective column replication, and the limitations of each approach.

## Key Concepts

- **Row Filter**: A WHERE clause on a publication that restricts which rows are replicated based on column values.
- **Column List**: An explicit list of columns to include in a publication, excluding unlisted columns from replication.
- **REPLICA IDENTITY**: For UPDATE and DELETE operations with row filters, the filtered column must be included in the table's replica identity.
- **IMMUTABLE Functions**: Only immutable functions (whose output depends solely on input) can be used in row filter expressions.

## Real World Context

A European company must comply with GDPR by not replicating personal data to servers outside the EU. Using column filtering, they exclude `email`, `phone`, and `address` columns from the publication feeding the US analytics database. Row filtering further limits the data to only records where `gdpr_consent = true`, ensuring compliance at the replication level.

## Deep Dive

Row filters use WHERE clauses to replicate only matching rows:

```sql
-- Only replicate active users
CREATE PUBLICATION active_users FOR TABLE users
    WHERE (status = 'active');

-- Only replicate orders from specific region
CREATE PUBLICATION us_orders FOR TABLE orders
    WHERE (region = 'US' AND amount > 100);

-- Multiple tables with different filters
CREATE PUBLICATION regional FOR TABLE
    orders WHERE (region = 'EU'),
    customers WHERE (country IN ('DE', 'FR', 'IT'));
```

Each table in a publication can have its own independent row filter.

Row filters have important limitations:

- Only simple expressions allowed (no subqueries, aggregates)
- Functions must be IMMUTABLE
- Cannot reference system columns
- UPDATE/DELETE only work if the filter column is in REPLICA IDENTITY

```sql
-- Set REPLICA IDENTITY to include filter column
ALTER TABLE orders REPLICA IDENTITY USING INDEX orders_region_idx;
```

This ensures the region column value is included in WAL records for UPDATE and DELETE operations.

Column filtering restricts which columns are replicated:

```sql
-- Replicate only specific columns
CREATE PUBLICATION partial_users FOR TABLE users (id, email, name);

-- Exclude sensitive columns (password_hash, ssn are NOT replicated)
CREATE PUBLICATION safe_users FOR TABLE users (id, name, email, created_at);
```

Only the listed columns appear in the replicated data. Unlisted columns receive their default values on the subscriber.

You can combine row and column filters:

```sql
-- Replicate specific columns for specific rows
CREATE PUBLICATION filtered FOR TABLE
    users (id, name, email) WHERE (is_public = true);
```

Primary key columns are always included even if not explicitly listed, because logical replication needs them to identify rows.

## Common Pitfalls

- **Using mutable functions in row filters**: Functions like `now()` or `random()` are not allowed in row filters because their output varies across executions.
- **Forgetting to update REPLICA IDENTITY**: Without the filtered column in the replica identity, UPDATE and DELETE operations for filtered rows will fail during decoding.
- **Assuming excluded columns are NULL on subscriber**: Excluded columns get their default value on the subscriber, which may be NULL or a configured default.

## Best Practices

- Always set REPLICA IDENTITY to include columns used in row filters when UPDATE or DELETE operations are expected.
- Use column filtering for privacy compliance to prevent sensitive data from reaching non-production environments.
- Test filter combinations thoroughly before deploying to production to ensure all expected rows are captured.

## Summary

- Row filters use WHERE clauses to selectively replicate rows matching specific conditions.
- Column lists restrict which columns are included in replicated data.
- Row and column filters can be combined for precise data distribution.
- REPLICA IDENTITY must include filtered columns for UPDATE and DELETE to work correctly.
- Primary key columns are always included regardless of column list specification.

## Code Examples

**Advanced Filtering with Row and Column Lists**

```sql
-- Multi-tenant filtering: only replicate tenant's data
CREATE PUBLICATION tenant_123 FOR TABLE
    orders WHERE (tenant_id = 123),
    customers WHERE (tenant_id = 123),
    products WHERE (tenant_id = 123);

-- Column filtering for privacy
CREATE PUBLICATION analytics FOR TABLE
    users (id, created_at, country, signup_source);  -- No PII

-- Combined filtering for GDPR compliance
CREATE PUBLICATION eu_customers FOR TABLE
    customers (id, email, country, created_at)
    WHERE (gdpr_consent = true AND country IN ('DE', 'FR', 'IT'));
```


## Resources

- [Row Filters](https://www.postgresql.org/docs/current/logical-replication-row-filter.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*