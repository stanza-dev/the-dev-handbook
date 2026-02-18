---
source_course: "postgresql-security"
source_lesson: "postgresql-security-ownership"
---

# Object Ownership

Every database object has an owner who has full control over it.

## Default Ownership

```sql
-- Objects are owned by the role that creates them
CREATE TABLE orders (id SERIAL PRIMARY KEY);
-- Owner is current user

-- Check ownership
SELECT tableowner FROM pg_tables WHERE tablename = 'orders';
```

## Changing Ownership

```sql
-- Transfer ownership of one object
ALTER TABLE orders OWNER TO admin;

-- Transfer all objects in schema
ALTER SCHEMA sales OWNER TO admin;

-- Transfer all objects owned by a role
REASSIGN OWNED BY alice TO bob;
```

## Owner Privileges

The owner can always:
- Grant/revoke privileges on the object
- Drop the object
- Alter the object
- Transfer ownership

```sql
-- Even without explicit GRANT, owner has all privileges
CREATE TABLE my_table (data TEXT);
-- Creator can SELECT, INSERT, UPDATE, DELETE, etc.
```

## Schema Search Path

Controls which schema is used for unqualified names:

```sql
-- Check current search path
SHOW search_path;

-- Set for session
SET search_path TO myschema, public;

-- Set for role
ALTER ROLE app_user SET search_path TO app_schema, public;
```

## Security Best Practices

```sql
-- 1. Create separate schema for application
CREATE SCHEMA app AUTHORIZATION app_owner;

-- 2. Revoke default public access
REVOKE ALL ON SCHEMA public FROM PUBLIC;
REVOKE CREATE ON SCHEMA public FROM PUBLIC;

-- 3. Grant specific access
GRANT USAGE ON SCHEMA app TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO app_user;
```

## Role Hierarchy Example

```sql
-- Create role hierarchy
CREATE ROLE readonly;
CREATE ROLE readwrite;
CREATE ROLE admin;

-- Build hierarchy
GRANT readonly TO readwrite;
GRANT readwrite TO admin;

-- Grant privileges to groups
GRANT SELECT ON ALL TABLES IN SCHEMA app TO readonly;
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO readwrite;
GRANT TRUNCATE ON ALL TABLES IN SCHEMA app TO admin;

-- Users inherit from their group
CREATE ROLE alice WITH LOGIN PASSWORD 'pass';
GRANT readwrite TO alice;  -- alice can SELECT, INSERT, UPDATE, DELETE
```

📖 [Schemas](https://www.postgresql.org/docs/18/ddl-schemas.html)

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*