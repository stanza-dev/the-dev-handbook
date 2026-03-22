---
source_course: "postgresql-security"
source_lesson: "postgresql-security-rls-debugging"
---

# RLS Debugging and Performance

## Introduction
RLS policies are powerful but can be tricky to debug when rows unexpectedly disappear or performance degrades. This lesson covers practical techniques for diagnosing RLS issues, understanding query plan impacts, and optimizing policies for production workloads.

## Key Concepts
- **Policy Inspection**: System catalogs and commands for viewing active RLS policies.
- **EXPLAIN ANALYZE**: Understanding how RLS policies appear in query plans and their performance impact.
- **Index Optimization**: Ensuring columns used in RLS policies are properly indexed.

## Real World Context
A developer reports that a SELECT query returns zero rows even though data exists. The issue turns out to be an RLS policy using `current_setting('app.tenant_id')` but the application never set that variable, causing a NULL comparison that matches nothing. Systematic debugging techniques would have caught this in minutes rather than hours.

## Deep Dive

### Inspecting Active Policies

```sql
-- View all policies on a table
SELECT polname, polpermissive, polroles::regrole[],
       polcmd, polqual, polwithcheck
FROM pg_policy
WHERE polrelid = 'orders'::regclass;

-- Or use psql shorthand
\d orders
```

### Debugging Unexpected Results

When rows seem to disappear, check:

```sql
-- 1. Verify RLS is enabled
SELECT relname, relrowsecurity, relforcerowsecurity
FROM pg_class WHERE relname = 'orders';

-- 2. Check current role
SELECT current_user, current_setting('app.tenant_id', true);

-- 3. Temporarily bypass RLS to see all data (as superuser)
SET ROLE postgres;
SELECT count(*) FROM orders;
RESET ROLE;

-- 4. Test specific policy conditions
SELECT *, (tenant_id = current_setting('app.tenant_id')::int) AS policy_match
FROM orders;  -- Run as superuser to see all rows
```

### Performance Impact and EXPLAIN

RLS policies add filter conditions to every query. Use EXPLAIN to see them:

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders WHERE status = 'pending';

-- Look for "Filter: (tenant_id = ...)" added by RLS
-- If it's a Seq Scan, you need an index
```

### Index Optimization

Always index columns used in RLS policies:

```sql
-- For tenant-based RLS
CREATE INDEX idx_orders_tenant_id ON orders(tenant_id);

-- For user-based RLS
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Composite index for RLS + common queries
CREATE INDEX idx_orders_tenant_status ON orders(tenant_id, status);
```

### Common Debugging Patterns

```sql
-- Check if a specific user can see a specific row
SET ROLE app_user;
SET app.tenant_id = '42';
SELECT * FROM orders WHERE id = 123;
-- If empty, the RLS policy is filtering it out
RESET ROLE;
```

## Common Pitfalls
1. **Not indexing RLS policy columns** — RLS adds a filter to every query. Without an index on the filtered column, every query becomes a sequential scan.
2. **Using volatile functions in policies** — Functions like `now()` or custom functions that are VOLATILE prevent the optimizer from using indexes effectively.

## Best Practices
1. **Index all columns referenced in RLS policies** — Treat RLS policy columns like any frequently-filtered column.
2. **Use EXPLAIN regularly** — Check that RLS-filtered queries use index scans, especially on large tables.

## Summary
- Use pg_policy and EXPLAIN ANALYZE to inspect and debug RLS behavior.
- Always index columns used in RLS policies to avoid sequential scans.
- Test policies with SET ROLE to verify correct access from different user perspectives.

## Code Examples

**Inspecting RLS policies and analyzing their impact on query plans**

```sql
-- Inspect policies on a table
SELECT polname, polpermissive, polcmd,
       pg_get_expr(polqual, polrelid) AS using_expr,
       pg_get_expr(polwithcheck, polrelid) AS check_expr
FROM pg_policy
WHERE polrelid = 'orders'::regclass;

-- Check query plan with RLS
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders WHERE status = 'pending';
```


## Resources

- [Row Security Policies](https://www.postgresql.org/docs/18/ddl-rowsecurity.html) — RLS documentation including performance considerations

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*