---
source_course: "postgresql-security"
source_lesson: "postgresql-security-schema-patterns"
---

# Multi-Tenant Implementation Patterns

## Introduction
Once you have chosen schema-per-tenant isolation, you need robust patterns for provisioning, querying, and managing tenant schemas at scale. This lesson covers automated provisioning, cross-schema admin queries, and the critical interaction between schema isolation and connection pooling.

## Key Concepts
- **Automated Provisioning**: A function that creates a complete tenant environment (schema, tables, roles, grants) in one call.
- **Cross-Schema Queries**: Admin-only queries that aggregate data across all tenant schemas.
- **Connection Pooling**: Using PgBouncer or pgpool-II with schema isolation requires careful session vs transaction mode handling.

## Real World Context
Your SaaS platform signs up 10 new customers per day. Manually creating schemas, tables, and roles would be error-prone and unscalable. An automated provisioning function ensures every tenant gets an identical, correctly-configured environment. Meanwhile, your operations team needs cross-tenant dashboards — which requires admin-level cross-schema queries.

## Deep Dive

### Automated Schema Provisioning

```sql
CREATE OR REPLACE FUNCTION create_tenant_schema(tenant_id TEXT)
RETURNS VOID AS $$
DECLARE
    schema_name TEXT := 'tenant_' || tenant_id;
BEGIN
    EXECUTE format('CREATE SCHEMA IF NOT EXISTS %I', schema_name);

    EXECUTE format('
        CREATE TABLE %I.users (
            id SERIAL PRIMARY KEY,
            email TEXT UNIQUE NOT NULL,
            name TEXT,
            created_at TIMESTAMPTZ DEFAULT NOW()
        )', schema_name);

    EXECUTE format('
        CREATE TABLE %I.orders (
            id SERIAL PRIMARY KEY,
            user_id INTEGER REFERENCES %I.users(id),
            total NUMERIC(10,2),
            status TEXT DEFAULT ''pending''
        )', schema_name, schema_name);

    EXECUTE format('CREATE ROLE %I WITH LOGIN PASSWORD %L',
                   tenant_id || '_user', gen_random_uuid()::text);

    EXECUTE format('GRANT USAGE ON SCHEMA %I TO %I',
                   schema_name, tenant_id || '_user');
    EXECUTE format('GRANT ALL ON ALL TABLES IN SCHEMA %I TO %I',
                   schema_name, tenant_id || '_user');
END;
$$ LANGUAGE plpgsql;

SELECT create_tenant_schema('newcorp');
```

### Cross-Schema Queries (Admin Only)

```sql
CREATE OR REPLACE FUNCTION get_all_tenant_user_counts()
RETURNS TABLE(tenant TEXT, user_count BIGINT) AS $$
DECLARE
    schema_name TEXT;
BEGIN
    FOR schema_name IN
        SELECT nspname FROM pg_namespace WHERE nspname LIKE 'tenant_%'
    LOOP
        RETURN QUERY EXECUTE format(
            'SELECT %L::text, COUNT(*)::bigint FROM %I.users',
            schema_name, schema_name
        );
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

### Connection Pooling with Schema Isolation

```sql
-- With PgBouncer in transaction mode, use SET LOCAL
BEGIN;
SET LOCAL search_path TO tenant_acme, public;
-- All queries in this transaction use tenant_acme
COMMIT;
```

### Dropping a Tenant

```sql
CREATE OR REPLACE FUNCTION drop_tenant_schema(tenant_id TEXT)
RETURNS VOID AS $$
DECLARE
    schema_name TEXT := 'tenant_' || tenant_id;
    role_name TEXT := tenant_id || '_user';
BEGIN
    EXECUTE format('DROP SCHEMA IF EXISTS %I CASCADE', schema_name);
    EXECUTE format('DROP ROLE IF EXISTS %I', role_name);
END;
$$ LANGUAGE plpgsql;
```

## Common Pitfalls
1. **Using session-level SET with transaction pooling** — In PgBouncer's transaction mode, session-level `SET search_path` can bleed between transactions. Always use `SET LOCAL` inside a transaction.
2. **Not granting on future tables** — After provisioning, if you add new tables to a tenant schema, existing roles will not have access unless you also set default privileges.

## Best Practices
1. **Use a provisioning function** — Ensure identical schema structure and permissions for every tenant.
2. **Use SET LOCAL with connection poolers** — This limits the search_path change to the current transaction, preventing cross-tenant leakage.

## Summary
- Automate tenant provisioning with a PL/pgSQL function for consistency.
- Use SET LOCAL search_path when using connection poolers.
- Cross-schema queries should be restricted to admin roles only.

## Code Examples

**Using SET LOCAL for safe tenant isolation with PgBouncer transaction pooling**

```sql
-- Safe tenant context with connection pooling
BEGIN;
SET LOCAL search_path TO tenant_acme, public;
SELECT * FROM users;
SELECT * FROM orders;
COMMIT;
-- search_path reverts after COMMIT
```


## Resources

- [Schema Search Path](https://www.postgresql.org/docs/18/runtime-config-client.html#GUC-SEARCH-PATH) — search_path configuration reference

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*