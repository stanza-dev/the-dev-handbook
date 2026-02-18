---
source_course: "postgresql-security"
source_lesson: "postgresql-security-schema-isolation"
---

# Schema-Based Isolation

Schemas provide a powerful mechanism for organizing and isolating data in multi-tenant applications.

## Multi-Tenancy Approaches

| Approach | Isolation | Complexity | Use Case |
|----------|-----------|------------|----------|
| Shared tables | Low | Low | Small tenants, simple data |
| Schema per tenant | High | Medium | **Recommended** |
| Database per tenant | Highest | High | Strict compliance needs |

## Schema-Per-Tenant Pattern

```sql
-- Create schema for each tenant
CREATE SCHEMA tenant_acme;
CREATE SCHEMA tenant_globex;

-- Create identical tables in each schema
CREATE TABLE tenant_acme.users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    name TEXT
);

CREATE TABLE tenant_globex.users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    name TEXT
);
```

## Using search_path for Tenant Context

```sql
-- Set tenant context at connection time
SET search_path TO tenant_acme, public;

-- Now queries automatically use tenant's schema
SELECT * FROM users;  -- Uses tenant_acme.users
INSERT INTO orders (user_id, total) VALUES (1, 99.99);

-- Switch tenant
SET search_path TO tenant_globex, public;
SELECT * FROM users;  -- Uses tenant_globex.users
```

## Role-Based Schema Access

```sql
-- Create role for each tenant
CREATE ROLE acme_app WITH LOGIN PASSWORD 'secret';
CREATE ROLE globex_app WITH LOGIN PASSWORD 'secret';

-- Grant schema access
GRANT USAGE ON SCHEMA tenant_acme TO acme_app;
GRANT ALL ON ALL TABLES IN SCHEMA tenant_acme TO acme_app;

-- Set default search path per role
ALTER ROLE acme_app SET search_path TO tenant_acme, public;
```

## Dynamic Schema Selection

```sql
-- Function to set tenant context
CREATE OR REPLACE FUNCTION set_tenant(tenant_name TEXT)
RETURNS VOID AS $$
BEGIN
    EXECUTE format('SET search_path TO %I, public', 'tenant_' || tenant_name);
END;
$$ LANGUAGE plpgsql;

-- Usage in application
SELECT set_tenant('acme');
```

📖 [Schemas](https://www.postgresql.org/docs/18/ddl-schemas.html)

## Resources

- [PostgreSQL Schemas](https://www.postgresql.org/docs/18/ddl-schemas.html) — Complete schema documentation

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*