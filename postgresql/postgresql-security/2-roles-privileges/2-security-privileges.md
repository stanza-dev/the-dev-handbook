---
source_course: "postgresql-security"
source_lesson: "postgresql-security-privileges"
---

# GRANT and REVOKE

## Introduction
Privileges control what operations roles can perform on database objects. PostgreSQL offers a granular privilege system covering tables, columns, schemas, databases, functions, and more. The `GRANT` and `REVOKE` commands are the primary tools for managing these permissions.

## Key Concepts
- **Object Privilege**: Permission to perform a specific operation (SELECT, INSERT, UPDATE, DELETE, etc.) on a specific database object.
- **WITH GRANT OPTION**: Allows the grantee to pass the privilege on to other roles.
- **DEFAULT PRIVILEGES**: Automatically apply grants to objects created in the future.
- **pg_get_acl()**: A new function in PostgreSQL 18 for programmatically retrieving the access control list (ACL) of a database object, taking the catalog OID, object OID, and sub-object ID as arguments.

## Real World Context
A typical web application has multiple services: the main app needs read-write access, the reporting service needs read-only access, and the migration runner needs DDL privileges. Using grants and group roles, you can ensure each service has exactly the permissions it needs — nothing more — following the principle of least privilege.

## Deep Dive

### Table Privileges

```sql
GRANT SELECT ON products TO analysts;
GRANT SELECT, INSERT, UPDATE ON orders TO app_user;
GRANT ALL PRIVILEGES ON customers TO admin;

-- Grant on all tables in schema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO reporting;

-- Revoke
REVOKE INSERT ON orders FROM app_user;
```

### Privilege Types

| Privilege | Allows |
|-----------|--------|
| `SELECT` | Read rows |
| `INSERT` | Add rows |
| `UPDATE` | Modify rows |
| `DELETE` | Remove rows |
| `TRUNCATE` | Empty table |
| `REFERENCES` | Create foreign keys |
| `TRIGGER` | Create triggers |
| `ALL` | All of the above |

### Column-Level Privileges

```sql
GRANT UPDATE (status, updated_at) ON orders TO app_user;
GRANT SELECT (id, name, email) ON users TO support_team;
```

### Schema and Database Privileges

```sql
GRANT USAGE ON SCHEMA sales TO analysts;
GRANT CREATE ON SCHEMA sales TO developers;
GRANT CONNECT ON DATABASE myapp TO app_user;
```

### Default Privileges

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE admin IN SCHEMA public
    GRANT SELECT ON TABLES TO analysts;
```

### Retrieving ACLs with pg_get_acl() (PostgreSQL 18)

PostgreSQL 18 introduces `pg_get_acl()` for programmatically retrieving access control lists:

```sql
-- Get ACL for a table using pg_get_acl()
SELECT pg_get_acl('pg_class'::regclass, 'orders'::regclass, 0);
```

The three arguments are: the catalog containing the object (`pg_class` for tables), the object itself, and the sub-object ID (0 for the whole object, or a column number for column-level ACLs). This function returns a structured representation of all grants, making it easier to audit permissions programmatically compared to parsing the `relacl` column.

### Checking Privileges

```sql
SELECT * FROM information_schema.table_privileges
WHERE table_name = 'orders';
```

## Common Pitfalls
1. **Granting ALL PRIVILEGES when only SELECT is needed** — Over-privileging roles violates least privilege and expands the blast radius of a compromised credential.
2. **Forgetting schema USAGE grants** — A role needs `USAGE` on a schema before it can access any objects within it. Table-level grants alone are not sufficient.

## Best Practices
1. **Use DEFAULT PRIVILEGES for consistency** — Set up default privileges so that new tables automatically inherit the correct permission structure.
2. **Use `pg_get_acl()` for auditing (PG 18)** — Regularly review ACLs programmatically to catch privilege drift.

## Summary
- PostgreSQL supports granular privileges at table, column, schema, database, and function levels.
- Default privileges ensure consistent permissions for future objects.
- PostgreSQL 18 adds `pg_get_acl()` for easier programmatic permission auditing.

## Code Examples

**Configuring default privileges and auditing existing grants**

```sql
-- Set up default privileges for a team
ALTER DEFAULT PRIVILEGES FOR ROLE admin IN SCHEMA public
    GRANT SELECT ON TABLES TO analysts;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT EXECUTE ON FUNCTIONS TO app_user;

-- Check existing privileges
SELECT grantee, table_name, privilege_type
FROM information_schema.table_privileges
WHERE table_name = 'orders';
```


## Resources

- [Privileges](https://www.postgresql.org/docs/18/ddl-priv.html) — Privilege system overview

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*