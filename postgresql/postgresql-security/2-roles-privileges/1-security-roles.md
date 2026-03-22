---
source_course: "postgresql-security"
source_lesson: "postgresql-security-roles"
---

# Understanding Roles

## Introduction
In PostgreSQL, roles are the foundation of access control. A role can represent a user, a group, or both. Unlike many databases that separate "users" and "groups," PostgreSQL unifies them into a single concept, giving you powerful flexibility in designing permission hierarchies.

## Key Concepts
- **Login Role**: A role with the `LOGIN` attribute — effectively a "user" that can connect to the database.
- **Group Role**: A role without `LOGIN` that serves as a container for privileges, granted to other roles.
- **INHERIT vs NOINHERIT**: Controls whether a role automatically receives the privileges of roles it belongs to, or must explicitly `SET ROLE`.
- **pg_signal_autovacuum_worker**: A new predefined role in PostgreSQL 18 that allows signaling autovacuum workers for maintenance purposes.

## Real World Context
Consider an application with three tiers of access: read-only analysts, read-write application users, and administrators. Rather than granting privileges to each individual, you create group roles (`readonly`, `readwrite`, `admin`) and then add users to the appropriate group. When a new analyst joins, you simply `GRANT readonly TO new_analyst` — no per-table grants needed.

## Deep Dive

### Creating Roles

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
    VALID UNTIL '2027-12-31';
```

### Role Attributes

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

PostgreSQL 18 also introduces the `pg_signal_autovacuum_worker` predefined role, which grants the ability to send signals to autovacuum worker processes — useful for DBAs who need to manage autovacuum without full superuser access.

### Role Membership (Groups)

```sql
CREATE ROLE developers;
GRANT developers TO alice;
GRANT developers TO bob;

-- View memberships
SELECT r.rolname AS role, m.rolname AS member
FROM pg_roles r
JOIN pg_auth_members am ON r.oid = am.roleid
JOIN pg_roles m ON am.member = m.oid
WHERE r.rolname = 'developers';
```

### INHERIT vs NOINHERIT

```sql
-- With INHERIT (default): automatically gets group privileges
CREATE ROLE alice WITH LOGIN INHERIT;
GRANT developers TO alice;

-- With NOINHERIT: must explicitly SET ROLE
CREATE ROLE bob WITH LOGIN NOINHERIT;
GRANT admin TO bob;
-- bob must: SET ROLE admin; to use admin privileges
```

### Dropping Roles Safely

```sql
REASSIGN OWNED BY alice TO postgres;
DROP OWNED BY alice;
DROP ROLE alice;
```

## Common Pitfalls
1. **Using superuser for applications** — Superuser bypasses all permission checks including RLS. Always create a least-privilege role for application connections.
2. **Forgetting REASSIGN before DROP** — You cannot drop a role that owns objects. Always reassign ownership first.

## Best Practices
1. **Use group roles for permission management** — Grant privileges to group roles, then add users to groups. This scales much better than per-user grants.
2. **Use NOINHERIT for elevated roles** — Force users to explicitly `SET ROLE` before using admin privileges, creating a clear audit trail.

## Summary
- Roles unify users and groups in PostgreSQL — a role with `LOGIN` is a user.
- Use group roles and membership to build scalable permission hierarchies.
- PostgreSQL 18 adds `pg_signal_autovacuum_worker` as a new predefined role for autovacuum management.

## Code Examples

**Creating a role hierarchy with cascading privileges**

```sql
-- Build a role hierarchy
CREATE ROLE readonly;
CREATE ROLE readwrite;
CREATE ROLE admin;

GRANT readonly TO readwrite;
GRANT readwrite TO admin;

-- Grant privileges to groups
GRANT SELECT ON ALL TABLES IN SCHEMA app TO readonly;
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO readwrite;

-- Add a user
CREATE ROLE alice WITH LOGIN PASSWORD 'pass';
GRANT readwrite TO alice;
```


## Resources

- [Database Roles](https://www.postgresql.org/docs/18/database-roles.html) — Role management guide

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*