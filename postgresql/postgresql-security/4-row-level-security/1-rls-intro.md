---
source_course: "postgresql-security"
source_lesson: "postgresql-security-rls-intro"
---

# Introduction to Row-Level Security

## Introduction
Row-Level Security (RLS) lets you control which rows individual users can see or modify, directly at the database level. Instead of relying on application code to filter data, RLS policies enforce access rules automatically — even if a query forgets to include a WHERE clause. This is one of PostgreSQL's most powerful security features.

## Key Concepts
- **RLS Policy**: A rule attached to a table that defines which rows a role can access for a given operation (SELECT, INSERT, UPDATE, DELETE).
- **ENABLE ROW LEVEL SECURITY**: Activates RLS on a table. Without policies, all rows are denied by default.
- **FORCE ROW LEVEL SECURITY**: Applies RLS even to the table owner (who normally bypasses it).
- **BYPASSRLS**: A role attribute that exempts a role from all RLS policies.

## Real World Context
Consider a multi-tenant SaaS application where all tenants share the same `orders` table. Without RLS, every query must include `WHERE tenant_id = ?` — and a single missed filter leaks data. With RLS, a policy enforces `tenant_id = current_setting('app.tenant_id')` automatically, making data leakage impossible at the database layer.

## Deep Dive

### Why RLS?

The traditional approach is fragile:

```sql
-- Must remember this filter in EVERY query
SELECT * FROM orders WHERE user_id = current_user_id();
```

With RLS, the filter is automatic:

```sql
SELECT * FROM orders;  -- Only sees their own orders
```

### Enabling RLS

```sql
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    owner TEXT NOT NULL,
    content TEXT,
    is_public BOOLEAN DEFAULT false
);

-- Enable RLS (without policies, denies all access!)
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Force RLS even for table owner
ALTER TABLE documents FORCE ROW LEVEL SECURITY;
```

### Creating Policies

```sql
CREATE POLICY user_sees_own_docs ON documents
    FOR SELECT
    USING (owner = current_user);

CREATE POLICY public_docs_visible ON documents
    FOR SELECT
    USING (is_public = true);
```

### How Policies Combine

Multiple PERMISSIVE policies for the same command type are combined with OR:

```sql
-- User sees docs where:
-- (owner = current_user) OR (is_public = true)
```

### Policy Syntax

```sql
CREATE POLICY name ON table
    [AS {PERMISSIVE | RESTRICTIVE}]
    [FOR {ALL | SELECT | INSERT | UPDATE | DELETE}]
    [TO role_name]
    [USING (expression)]          -- Filter existing rows
    [WITH CHECK (expression)];    -- Validate new/modified rows
```

## Common Pitfalls
1. **Enabling RLS without creating policies** — This denies all access to the table (except for the owner and superusers). Always create at least one policy before enabling RLS on a table with active users.
2. **Forgetting FORCE ROW LEVEL SECURITY** — Without this, the table owner bypasses all RLS policies, which can be a security gap.

## Best Practices
1. **Always use FORCE ROW LEVEL SECURITY** — This ensures even the table owner is subject to policies, preventing accidental full-table access.
2. **Test RLS policies with SET ROLE** — Use `SET ROLE app_user` in development to verify that policies correctly restrict access.

## Summary
- RLS enforces row-level access at the database layer, eliminating reliance on application-level filtering.
- Enable RLS with `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` and always use FORCE.
- Without policies, RLS denies all access by default — always create policies before enabling.

## Code Examples

**Enabling RLS, creating a policy, and testing with role switching**

```sql
-- Enable RLS and create a policy
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents FORCE ROW LEVEL SECURITY;

CREATE POLICY user_sees_own ON documents
    FOR SELECT
    USING (owner = current_user);

-- Test with SET ROLE
SET ROLE app_user;
SELECT * FROM documents;  -- Only sees own docs
RESET ROLE;
```


## Resources

- [Row Security Policies](https://www.postgresql.org/docs/18/ddl-rowsecurity.html) — Complete RLS documentation

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*