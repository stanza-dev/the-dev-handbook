---
source_course: "postgresql-security"
source_lesson: "postgresql-security-rls-policies"
---

# Writing RLS Policies

## Introduction
Writing effective RLS policies requires understanding the difference between USING and WITH CHECK clauses, how PERMISSIVE and RESTRICTIVE policies combine, and how to target policies to specific roles. This lesson covers all of these patterns with practical examples.

## Key Concepts
- **USING clause**: Filters which existing rows are visible for SELECT, UPDATE, and DELETE operations.
- **WITH CHECK clause**: Validates new or modified rows for INSERT and UPDATE operations.
- **PERMISSIVE vs RESTRICTIVE**: PERMISSIVE policies OR together; RESTRICTIVE policies AND with the combined PERMISSIVE result.

## Real World Context
A healthcare application must enforce HIPAA-compliant data isolation: doctors see only their patients' records, nurses see records for their department, and admins see everything. This requires role-specific policies with both USING (what they can read) and WITH CHECK (what they can write) clauses, plus a RESTRICTIVE policy ensuring only active records are ever visible.

## Deep Dive

### USING vs WITH CHECK

```sql
-- Users can only see their own orders
CREATE POLICY see_own_orders ON orders
    FOR SELECT
    USING (user_id = current_user_id());

-- Users can only insert orders for themselves
CREATE POLICY insert_own_orders ON orders
    FOR INSERT
    WITH CHECK (user_id = current_user_id());

-- Combined for UPDATE: can only update own orders, must remain own
CREATE POLICY update_own_orders ON orders
    FOR UPDATE
    USING (user_id = current_user_id())
    WITH CHECK (user_id = current_user_id());
```

### Multi-Tenant Isolation with Application Context

```sql
CREATE TABLE tenant_data (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL,
    data TEXT
);

ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON tenant_data
    FOR ALL
    USING (tenant_id = current_setting('app.tenant_id')::INTEGER)
    WITH CHECK (tenant_id = current_setting('app.tenant_id')::INTEGER);

SET app.tenant_id = '42';
SELECT * FROM tenant_data;  -- Only sees tenant 42's data
```

### Role-Specific Policies

```sql
CREATE POLICY admin_all ON orders
    FOR ALL TO admin_role
    USING (true);

CREATE POLICY user_own_orders ON orders
    FOR ALL TO app_user
    USING (user_id = current_user_id())
    WITH CHECK (user_id = current_user_id());
```

### PERMISSIVE vs RESTRICTIVE

```sql
-- PERMISSIVE (default): policies OR together
CREATE POLICY allow_own ON docs FOR SELECT
    USING (owner = current_user);

CREATE POLICY allow_public ON docs FOR SELECT
    USING (is_public = true);
-- Result: (owner = current_user) OR (is_public = true)

-- RESTRICTIVE: must pass along with permissive policies
CREATE POLICY active_only ON docs FOR SELECT
    AS RESTRICTIVE
    USING (is_active = true);
-- Result: ((owner = current_user) OR (is_public = true)) AND (is_active = true)
```

## Common Pitfalls
1. **Forgetting WITH CHECK on INSERT/UPDATE** — Without WITH CHECK, users might be able to insert rows they cannot later see, causing confusion.
2. **Circular dependencies in recursive policies** — Policies that reference the same table in subqueries can cause infinite loops or poor performance.

## Best Practices
1. **Pair USING with WITH CHECK for UPDATE policies** — Ensure users can only modify rows they can see, and the modified result remains visible to them.
2. **Use RESTRICTIVE policies for cross-cutting concerns** — Apply RESTRICTIVE policies for conditions like `is_active = true` that must always hold, regardless of other permissive policies.

## Summary
- USING filters readable/updatable/deletable rows; WITH CHECK validates new/modified rows.
- PERMISSIVE policies OR together; RESTRICTIVE policies AND with the permissive result.
- Role-specific policies let you define different access levels for different user types.

## Code Examples

**Implementing multi-tenant isolation with application context variables**

```sql
-- Multi-tenant RLS with application context
ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON tenant_data
    FOR ALL
    USING (tenant_id = current_setting('app.tenant_id')::INTEGER)
    WITH CHECK (tenant_id = current_setting('app.tenant_id')::INTEGER);

-- Set tenant context per connection
SET app.tenant_id = '42';
SELECT * FROM tenant_data;  -- Only tenant 42
```


## Resources

- [CREATE POLICY](https://www.postgresql.org/docs/18/sql-createpolicy.html) — Complete CREATE POLICY syntax reference

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*