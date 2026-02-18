---
source_course: "postgresql-security"
source_lesson: "postgresql-security-compliance"
---

# Security Best Practices

Harden your PostgreSQL installation for production.

## Authentication Checklist

```conf
# pg_hba.conf
# ❌ Never in production:
host all all 0.0.0.0/0 trust

# ✅ Good:
hostssl all all 0.0.0.0/0 scram-sha-256
```

## Network Security

```ini
# postgresql.conf

# Listen only where needed
listen_addresses = 'localhost'  # Or specific IPs
# listen_addresses = '*'  # Only if behind firewall

# Set port (optional obfuscation)
port = 5432
```

## Privilege Principles

1. **Least Privilege**: Grant minimum necessary permissions
2. **Separation of Duties**: Different roles for different tasks
3. **No Shared Accounts**: Individual accounts for each user

```sql
-- ✅ Good: Specific privileges
GRANT SELECT ON reports TO analyst;

-- ❌ Bad: Excessive privileges
GRANT ALL ON SCHEMA public TO analyst;
```

## Superuser Restrictions

```sql
-- Limit superuser usage
-- Use for administration only, not applications

-- Create admin role for most tasks
CREATE ROLE db_admin WITH CREATEDB CREATEROLE LOGIN PASSWORD 'secure';

-- Reserve superuser for emergencies
```

## Regular Security Reviews

```sql
-- Check for users with superuser
SELECT rolname FROM pg_roles WHERE rolsuper;

-- Check for roles that can bypass RLS
SELECT rolname FROM pg_roles WHERE rolbypassrls;

-- Find public grants
SELECT 
    nspname || '.' || relname AS object,
    array_agg(privilege_type) AS public_privileges
FROM information_schema.table_privileges tp
JOIN pg_class c ON c.relname = tp.table_name
JOIN pg_namespace n ON n.oid = c.relnamespace AND n.nspname = tp.table_schema
WHERE grantee = 'PUBLIC'
GROUP BY nspname, relname;
```

## Password Policies

```sql
-- Require password expiration
ALTER ROLE app_user VALID UNTIL '2025-06-01';

-- Connection limits
ALTER ROLE app_user CONNECTION LIMIT 10;
```

## Security Extensions

```sql
-- sepgsql: SELinux integration
-- anon: Data anonymization
-- pg_permissions: Permission reporting

CREATE EXTENSION pg_permissions;
SELECT * FROM all_permissions();
```

## Compliance Considerations

| Regulation | Key Requirements |
|------------|------------------|
| GDPR | Data encryption, access logging, right to erasure |
| HIPAA | Audit controls, access controls, encryption |
| PCI-DSS | Network security, encryption, access control |
| SOC 2 | Security monitoring, access management |

📖 [Server Security](https://www.postgresql.org/docs/18/auth-pg-hba-conf.html)

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*