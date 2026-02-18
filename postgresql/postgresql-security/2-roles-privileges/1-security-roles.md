---
source_course: "postgresql-security"
source_lesson: "postgresql-security-roles"
---

# Understanding Roles

In PostgreSQL, roles are the foundation of access control. A role can be a user, a group, or both.

## Creating Roles

```sql
-- Basic role (no login)
CREATE ROLE analysts;

-- Role with login (a "user")
CREATE ROLE alice WITH LOGIN PASSWORD 'secure_pass';

-- Role with multiple attributes
CREATE ROLE admin WITH
    LOGIN
    PASSWORD 'admin_pass'
    CREATEDB
    CREATEROLE
    VALID UNTIL '2025-12-31';
```

## Role Attributes

| Attribute | Description |
|-----------|-------------|
| `LOGIN` | Can connect to database |
| `SUPERUSER` | Bypass all permission checks |
| `CREATEDB` | Can create databases |
| `CREATEROLE` | Can create/alter roles |
| `REPLICATION` | Can initiate replication |
| `BYPASSRLS` | Bypass row-level security |
| `INHERIT` | Inherit privileges from member roles |
| `CONNECTION LIMIT n` | Max concurrent connections |

```sql
-- View role attributes
SELECT * FROM pg_roles WHERE rolname = 'alice';

-- Or use \du in psql
\du alice
```

## Role Membership (Groups)

```sql
-- Create group role
CREATE ROLE developers;

-- Add users to group
GRANT developers TO alice;
GRANT developers TO bob;

-- View memberships
SELECT r.rolname AS role, m.rolname AS member
FROM pg_roles r
JOIN pg_auth_members am ON r.oid = am.roleid
JOIN pg_roles m ON am.member = m.oid
WHERE r.rolname = 'developers';
```

## Role Inheritance

```sql
-- With INHERIT (default): automatically gets group privileges
CREATE ROLE alice WITH LOGIN INHERIT;
GRANT developers TO alice;
-- alice automatically has developers' privileges

-- With NOINHERIT: must explicitly SET ROLE
CREATE ROLE bob WITH LOGIN NOINHERIT;
GRANT admin TO bob;
-- bob must: SET ROLE admin; to use admin privileges
```

## Modifying Roles

```sql
-- Add attribute
ALTER ROLE alice WITH CREATEDB;

-- Remove attribute
ALTER ROLE alice WITH NOCREATEDB;

-- Change password
ALTER ROLE alice WITH PASSWORD 'new_password';

-- Rename role
ALTER ROLE alice RENAME TO alice_smith;
```

## Dropping Roles

```sql
-- First, revoke ownership and privileges
REASSIGN OWNED BY alice TO postgres;
DROP OWNED BY alice;

-- Then drop the role
DROP ROLE alice;
```

📖 [Database Roles](https://www.postgresql.org/docs/18/database-roles.html)

## Resources

- [Database Roles](https://www.postgresql.org/docs/18/database-roles.html) — Role management guide

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*