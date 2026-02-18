---
source_course: "postgresql-security"
source_lesson: "postgresql-security-privileges"
---

# GRANT and REVOKE

Privileges control what operations roles can perform on database objects.

## Table Privileges

```sql
-- Grant specific privileges
GRANT SELECT ON products TO analysts;
GRANT SELECT, INSERT, UPDATE ON orders TO app_user;
GRANT ALL PRIVILEGES ON customers TO admin;

-- Grant on all tables in schema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO reporting;

-- Revoke privileges
REVOKE INSERT ON orders FROM app_user;
REVOKE ALL PRIVILEGES ON customers FROM alice;
```

## Privilege Types

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

## Column-Level Privileges

```sql
-- Allow updates only on specific columns
GRANT UPDATE (status, updated_at) ON orders TO app_user;

-- Restrict SELECT to certain columns
GRANT SELECT (id, name, email) ON users TO support_team;
-- support_team cannot see password_hash, etc.
```

## Schema Privileges

```sql
-- Access to schema (required to see objects)
GRANT USAGE ON SCHEMA sales TO analysts;

-- Create objects in schema
GRANT CREATE ON SCHEMA sales TO developers;

-- Both
GRANT USAGE, CREATE ON SCHEMA sales TO admin;
```

## Database Privileges

```sql
-- Connect to database
GRANT CONNECT ON DATABASE myapp TO app_user;

-- Create schemas
GRANT CREATE ON DATABASE myapp TO admin;
```

## Function Privileges

```sql
-- Allow execution
GRANT EXECUTE ON FUNCTION calculate_total(integer) TO app_user;

-- All functions in schema
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO app_user;
```

## Default Privileges

Apply to future objects:

```sql
-- All future tables created by admin are readable by analysts
ALTER DEFAULT PRIVILEGES FOR ROLE admin IN SCHEMA public
    GRANT SELECT ON TABLES TO analysts;

-- All future functions are executable by app_user
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT EXECUTE ON FUNCTIONS TO app_user;
```

## WITH GRANT OPTION

```sql
-- Allow user to grant this privilege to others
GRANT SELECT ON products TO manager WITH GRANT OPTION;

-- manager can now:
GRANT SELECT ON products TO team_member;
```

## Checking Privileges

```sql
-- Check table privileges
SELECT * FROM information_schema.table_privileges
WHERE table_name = 'orders';

-- Or use \dp in psql
\dp orders
```

📖 [GRANT](https://www.postgresql.org/docs/18/sql-grant.html)

## Resources

- [Privileges](https://www.postgresql.org/docs/18/ddl-priv.html) — Privilege system overview

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*