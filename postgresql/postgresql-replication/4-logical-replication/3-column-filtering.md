---
source_course: "postgresql-replication"
source_lesson: "row-column-filtering"
---

# Row and Column Filtering

PostgreSQL 15+ supports filtering which rows and columns are replicated, enabling selective data distribution.

## Row Filtering

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

## Row Filter Limitations

- Only simple expressions allowed (no subqueries, aggregates)
- Functions must be IMMUTABLE
- Cannot reference system columns
- UPDATE/DELETE only work if filter column is in REPLICA IDENTITY

```sql
-- Set REPLICA IDENTITY to include filter column
ALTER TABLE orders REPLICA IDENTITY USING INDEX orders_region_idx;
```

## Column Filtering (Column Lists)

```sql
-- Replicate only specific columns
CREATE PUBLICATION partial_users FOR TABLE users (id, email, name);

-- Exclude sensitive columns
-- (password_hash, ssn are NOT replicated)
CREATE PUBLICATION safe_users FOR TABLE users (id, name, email, created_at);
```

## Combining Row and Column Filters

```sql
-- Replicate specific columns for specific rows
CREATE PUBLICATION filtered FOR TABLE 
    users (id, name, email) WHERE (is_public = true);
```

**Important:** Primary key columns are always included even if not in column list.

## Code Examples

```undefined
-- Multi-tenant filtering: only replicate tenant's data
CREATE PUBLICATION tenant_123 FOR TABLE 
    orders WHERE (tenant_id = 123),
    customers WHERE (tenant_id = 123),
    products WHERE (tenant_id = 123);

-- Column filtering for privacy
CREATE PUBLICATION analytics FOR TABLE 
    users (id, created_at, country, signup_source);  -- No PII

-- Combined filtering
CREATE PUBLICATION eu_customers FOR TABLE 
    customers (id, email, country, created_at) 
    WHERE (gdpr_consent = true AND country IN ('DE', 'FR', 'IT'));
```


## Resources

- [Row Filters](https://www.postgresql.org/docs/current/logical-replication-row-filter.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*