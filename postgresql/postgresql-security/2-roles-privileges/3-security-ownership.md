---
source_course: "postgresql-security"
source_lesson: "postgresql-security-ownership"
---

# Object Ownership

## Introduction
Every database object in PostgreSQL has an owner who has full control over it. Understanding ownership is essential because the owner can grant and revoke privileges, alter the object, and even drop it — regardless of other privilege settings. Ownership transfer is also a critical operation when decommissioning roles.

## Key Concepts
- **Owner**: The role that created an object. Has all privileges on it automatically.
- **REASSIGN OWNED**: Transfers ownership of all objects from one role to another.
- **search_path**: Controls which schema is used for unqualified object names — a common source of security issues.

## Real World Context
When an employee leaves your organization, you need to drop their database role. But if that role owns tables, views, or functions, PostgreSQL will refuse to drop it. The correct workflow is to first `REASSIGN OWNED BY departing_user TO dba`, then `DROP OWNED BY departing_user`, and finally `DROP ROLE departing_user`. Skipping this leads to orphaned objects or blocked role removal.

## Deep Dive

### Default Ownership

```sql
CREATE TABLE orders (id SERIAL PRIMARY KEY);
-- Owner is the current user who ran this statement

SELECT tableowner FROM pg_tables WHERE tablename = 'orders';
```

### Changing Ownership

```sql
ALTER TABLE orders OWNER TO admin;
ALTER SCHEMA sales OWNER TO admin;
REASSIGN OWNED BY alice TO bob;
```

### Schema Search Path Security

The `search_path` determines which schema is used for unqualified names:

```sql
SHOW search_path;
SET search_path TO myschema, public;
ALTER ROLE app_user SET search_path TO app_schema, public;
```

A common security pattern is to revoke default public schema access:

```sql
CREATE SCHEMA app AUTHORIZATION app_owner;
REVOKE ALL ON SCHEMA public FROM PUBLIC;
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
GRANT USAGE ON SCHEMA app TO app_user;
```

### Role Hierarchy Example

```sql
CREATE ROLE readonly;
CREATE ROLE readwrite;
CREATE ROLE admin;

GRANT readonly TO readwrite;
GRANT readwrite TO admin;

GRANT SELECT ON ALL TABLES IN SCHEMA app TO readonly;
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO readwrite;
GRANT TRUNCATE ON ALL TABLES IN SCHEMA app TO admin;

CREATE ROLE alice WITH LOGIN PASSWORD 'pass';
GRANT readwrite TO alice;
```

## Common Pitfalls
1. **Not revoking PUBLIC schema CREATE** — By default, any authenticated user can create objects in the `public` schema. This can lead to search_path hijacking attacks.
2. **Forgetting to REASSIGN before DROP ROLE** — PostgreSQL will refuse to drop a role that owns objects, leading to frustrating errors.

## Best Practices
1. **Revoke CREATE on public schema** — In PostgreSQL 15+, the default CREATE privilege on public was removed, but always verify with `REVOKE CREATE ON SCHEMA public FROM PUBLIC` — especially for databases upgraded from older versions.
2. **Set explicit search_path per role** — Avoid relying on the default search_path to prevent unintended schema resolution.

## Summary
- Object owners have full control over their objects, including granting privileges.
- Use REASSIGN OWNED to safely transfer ownership before dropping roles.
- Revoke default public schema privileges and set explicit search_path for security.

## Code Examples

**Safe role removal and secure schema configuration**

```sql
-- Safe role removal workflow
REASSIGN OWNED BY departing_user TO dba;
DROP OWNED BY departing_user;
DROP ROLE departing_user;

-- Secure schema setup
CREATE SCHEMA app AUTHORIZATION app_owner;
REVOKE ALL ON SCHEMA public FROM PUBLIC;
GRANT USAGE ON SCHEMA app TO app_user;
```


## Resources

- [Schemas](https://www.postgresql.org/docs/18/ddl-schemas.html) — Schema management and security

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*