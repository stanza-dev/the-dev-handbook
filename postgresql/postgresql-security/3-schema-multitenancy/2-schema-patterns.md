---
source_course: "postgresql-security"
source_lesson: "postgresql-security-schema-patterns"
---

# Multi-Tenant Implementation Patterns

Best practices for building schema-based multi-tenant systems.

## Automated Schema Provisioning

```sql
CREATE OR REPLACE FUNCTION create_tenant_schema(tenant_id TEXT)
RETURNS VOID AS $$
DECLARE
    schema_name TEXT := 'tenant_' || tenant_id;
BEGIN
    -- Create the schema
    EXECUTE format('CREATE SCHEMA IF NOT EXISTS %I', schema_name);
    
    -- Create tables from template
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
    
    -- Create tenant-specific role
    EXECUTE format('CREATE ROLE %I WITH LOGIN PASSWORD %L', 
                   tenant_id || '_user', gen_random_uuid()::text);
    
    -- Grant permissions
    EXECUTE format('GRANT USAGE ON SCHEMA %I TO %I', 
                   schema_name, tenant_id || '_user');
    EXECUTE format('GRANT ALL ON ALL TABLES IN SCHEMA %I TO %I', 
                   schema_name, tenant_id || '_user');
END;
$$ LANGUAGE plpgsql;

-- Create new tenant
SELECT create_tenant_schema('newcorp');
```

## Cross-Schema Queries (Admin Only)

```sql
-- Admin can query across all tenants
SELECT 
    nspname AS tenant,
    (SELECT COUNT(*) FROM pg_class c 
     WHERE c.relnamespace = n.oid AND c.relname = 'users') AS user_tables
FROM pg_namespace n
WHERE nspname LIKE 'tenant_%';

-- Aggregate data across tenants
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

## Connection Pooling Considerations

```sql
-- With connection poolers like PgBouncer, set search_path per transaction
BEGIN;
SET LOCAL search_path TO tenant_acme, public;
-- All queries in this transaction use tenant_acme
COMMIT;

-- Or use session-level configuration in connection string
-- postgresql://user:pass@host/db?options=-csearch_path=tenant_acme
```

## Dropping a Tenant

```sql
CREATE OR REPLACE FUNCTION drop_tenant_schema(tenant_id TEXT)
RETURNS VOID AS $$
DECLARE
    schema_name TEXT := 'tenant_' || tenant_id;
    role_name TEXT := tenant_id || '_user';
BEGIN
    -- Drop schema and all objects
    EXECUTE format('DROP SCHEMA IF EXISTS %I CASCADE', schema_name);
    
    -- Drop role
    EXECUTE format('DROP ROLE IF EXISTS %I', role_name);
END;
$$ LANGUAGE plpgsql;
```

📖 [Schema Search Path](https://www.postgresql.org/docs/18/runtime-config-client.html#GUC-SEARCH-PATH)

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*