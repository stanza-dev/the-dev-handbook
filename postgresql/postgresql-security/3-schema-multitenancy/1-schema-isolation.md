---
source_course: "postgresql-security"
source_lesson: "postgresql-security-schema-isolation"
---

# Schema Isolation Strategies

## Introduction
Schemas provide a powerful mechanism for organizing and isolating data in multi-tenant applications. Choosing the right isolation strategy determines your security boundaries, operational complexity, and ability to scale. This lesson compares the major approaches and deep-dives into the schema-per-tenant pattern.

## Key Concepts
- **Schema**: A namespace within a database that contains tables, views, functions, and other objects.
- **search_path**: A session variable that determines which schema is used for unqualified object names.
- **Schema-per-tenant**: An isolation pattern where each tenant gets a dedicated schema with identical table structures.

## Real World Context
A SaaS application serving 500 customers needs data isolation. Shared tables with a `tenant_id` column work but require WHERE clauses everywhere (and a single missed filter leaks data). Schema-per-tenant provides natural isolation: each tenant's queries hit their own schema, and a forgotten WHERE clause simply cannot access another tenant's data.

## Deep Dive

### Multi-Tenancy Approaches

| Approach | Isolation | Complexity | Use Case |
|----------|-----------|------------|----------|
| Shared tables | Low | Low | Small tenants, simple data |
| Schema per tenant | High | Medium | **Recommended for most SaaS** |
| Database per tenant | Highest | High | Strict compliance needs |

### Schema-Per-Tenant Pattern

```sql
CREATE SCHEMA tenant_acme;
CREATE SCHEMA tenant_globex;

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

### Using search_path for Tenant Context

```sql
SET search_path TO tenant_acme, public;
SELECT * FROM users;  -- Uses tenant_acme.users

SET search_path TO tenant_globex, public;
SELECT * FROM users;  -- Uses tenant_globex.users
```

### Role-Based Schema Access

```sql
CREATE ROLE acme_app WITH LOGIN PASSWORD 'secret';
GRANT USAGE ON SCHEMA tenant_acme TO acme_app;
GRANT ALL ON ALL TABLES IN SCHEMA tenant_acme TO acme_app;
ALTER ROLE acme_app SET search_path TO tenant_acme, public;
```

### Dynamic Schema Selection

```sql
CREATE OR REPLACE FUNCTION set_tenant(tenant_name TEXT)
RETURNS VOID AS $$
BEGIN
    EXECUTE format('SET search_path TO %I, public', 'tenant_' || tenant_name);
END;
$$ LANGUAGE plpgsql;

SELECT set_tenant('acme');
```

## Common Pitfalls
1. **Not restricting search_path per role** — If the application role can access any schema, a bug in tenant context selection can leak data between tenants.
2. **Forgetting to grant USAGE on the schema** — Table-level grants alone do not work without schema USAGE permission.

## Best Practices
1. **Lock down search_path per role** — Use `ALTER ROLE ... SET search_path` so each tenant role can only see its own schema.
2. **Automate schema provisioning** — Use a function to create schemas, tables, roles, and grants consistently for every new tenant.

## Summary
- Schema-per-tenant provides strong data isolation without per-query WHERE clauses.
- Use search_path and role-based access to enforce tenant boundaries.
- Automate schema creation to ensure consistency across hundreds of tenants.

## Code Examples

**Setting up a schema-per-tenant with dedicated role and locked search_path**

```sql
-- Create tenant schema with isolated role
CREATE SCHEMA tenant_acme;
CREATE ROLE acme_app WITH LOGIN PASSWORD 'secret';
GRANT USAGE ON SCHEMA tenant_acme TO acme_app;
GRANT ALL ON ALL TABLES IN SCHEMA tenant_acme TO acme_app;
ALTER ROLE acme_app SET search_path TO tenant_acme, public;
```


## Resources

- [PostgreSQL Schemas](https://www.postgresql.org/docs/18/ddl-schemas.html) — Complete schema documentation

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*