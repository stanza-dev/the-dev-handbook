---
source_course: "postgresql-security"
source_lesson: "postgresql-security-compliance"
---

# Security Best Practices

## Introduction
Hardening a PostgreSQL installation for production requires a systematic approach covering authentication, network security, privilege management, and regular audits. This lesson provides a comprehensive security checklist aligned with common compliance frameworks like GDPR, HIPAA, PCI-DSS, and SOC 2.

## Key Concepts
- **Least Privilege**: Grant the minimum permissions necessary for each role to perform its function.
- **Separation of Duties**: Different roles for different tasks — no single role should have all-encompassing access.
- **Defense in Depth**: Multiple layers of security so that a failure in one layer does not compromise the system.

## Real World Context
A PCI-DSS audit requires you to demonstrate: encrypted connections, individual user accounts (no shared passwords), audit logging, least-privilege access controls, and regular security reviews. A well-hardened PostgreSQL setup with SSL, pgAudit, role hierarchies, and documented access policies checks all these boxes.

## Deep Dive

### Authentication Checklist

```conf
# pg_hba.conf — NEVER in production:
host all all 0.0.0.0/0 trust

# GOOD:
hostssl all all 0.0.0.0/0 scram-sha-256
```

### Network Security

```ini
# postgresql.conf
listen_addresses = 'localhost'
port = 5432
```

### Privilege Principles

```sql
-- GOOD: Specific privileges
GRANT SELECT ON reports TO analyst;

-- BAD: Excessive privileges
GRANT ALL ON SCHEMA public TO analyst;
```

### Superuser Restrictions

```sql
CREATE ROLE db_admin WITH CREATEDB CREATEROLE LOGIN PASSWORD 'secure';
-- Reserve superuser for emergencies only
```

### Regular Security Reviews

```sql
-- Check for superusers
SELECT rolname FROM pg_roles WHERE rolsuper;

-- Check for RLS bypass
SELECT rolname FROM pg_roles WHERE rolbypassrls;

-- Find public grants
SELECT nspname || '.' || relname AS object,
    array_agg(privilege_type) AS public_privileges
FROM information_schema.table_privileges tp
JOIN pg_class c ON c.relname = tp.table_name
JOIN pg_namespace n ON n.oid = c.relnamespace AND n.nspname = tp.table_schema
WHERE grantee = 'PUBLIC'
GROUP BY nspname, relname;
```

### Password Policies

```sql
ALTER ROLE app_user VALID UNTIL '2027-06-01';
ALTER ROLE app_user CONNECTION LIMIT 10;
```

### Compliance Quick Reference

| Regulation | Key Requirements |
|------------|------------------|
| GDPR | Data encryption, access logging, right to erasure |
| HIPAA | Audit controls, access controls, encryption |
| PCI-DSS | Network security, encryption, access control |
| SOC 2 | Security monitoring, access management |

## Common Pitfalls
1. **Using superuser for application connections** — Superuser bypasses all security controls including RLS. Create dedicated application roles with minimal privileges.
2. **Not reviewing privileges regularly** — Privilege drift accumulates over time. Schedule quarterly reviews to remove unnecessary grants.

## Best Practices
1. **Implement least privilege from day one** — It is much harder to restrict privileges later than to start restrictive and expand as needed.
2. **Automate security reviews** — Create scripts that check for common issues (superuser count, public grants, expired passwords) and run them regularly.

## Summary
- Harden authentication with SCRAM-SHA-256 over SSL, and never use trust in production.
- Apply least privilege: specific grants to specific roles on specific objects.
- Schedule regular security reviews to catch privilege drift and compliance gaps.

## Code Examples

**Security audit queries for reviewing superusers, RLS bypass, expired passwords, and public grants**

```sql
-- Security audit queries
SELECT rolname FROM pg_roles WHERE rolsuper;
SELECT rolname FROM pg_roles WHERE rolbypassrls;
SELECT rolname, rolvaliduntil FROM pg_roles
WHERE rolvaliduntil IS NOT NULL AND rolvaliduntil < NOW();

-- Check for overly permissive public grants
SELECT table_schema, table_name, privilege_type
FROM information_schema.table_privileges
WHERE grantee = 'PUBLIC'
ORDER BY table_schema, table_name;
```


## Resources

- [Server Security](https://www.postgresql.org/docs/18/auth-pg-hba-conf.html) — Authentication and security configuration

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*