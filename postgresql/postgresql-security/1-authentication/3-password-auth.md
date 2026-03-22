---
source_course: "postgresql-security"
source_lesson: "postgresql-security-password-auth"
---

# Password Authentication

## Introduction
Password authentication is the most common way users connect to PostgreSQL. Getting it right means choosing the strongest available method (SCRAM-SHA-256), understanding encryption settings, and implementing password expiration policies. This lesson covers the full lifecycle of password-based auth in PostgreSQL 18.

## Key Concepts
- **SCRAM-SHA-256**: Salted Challenge Response Authentication Mechanism — the password never crosses the network in any form.
- **password_encryption**: The server setting that controls how new passwords are hashed and stored.
- **VALID UNTIL**: An optional role attribute that sets a password expiration date.

## Real World Context
A common production scenario: you inherit a database still using MD5 authentication. MD5 is officially deprecated in PostgreSQL 18, meaning future versions may remove it entirely. Migrating to SCRAM requires changing `password_encryption`, re-setting user passwords, and updating pg_hba.conf — a process that should be planned and executed during a maintenance window.

## Deep Dive

### SCRAM-SHA-256 (Recommended)

SCRAM provides strong security properties:
- Password never sent over the network
- Server does not store the plain password
- Protection against replay attacks
- Resistant to brute-force attacks

```conf
# pg_hba.conf
host    all    all    0.0.0.0/0    scram-sha-256
```

### Setting Password Encryption

```sql
SHOW password_encryption;

ALTER SYSTEM SET password_encryption = 'scram-sha-256';
SELECT pg_reload_conf();
```

### Creating Users with Passwords

```sql
CREATE ROLE app_user WITH LOGIN PASSWORD 'secure_password_here';

-- Check that the password is SCRAM-encrypted
SELECT rolname, rolpassword
FROM pg_authid
WHERE rolname = 'app_user';
-- SCRAM passwords start with 'SCRAM-SHA-256$'
```

### MD5 (Deprecated in PostgreSQL 18)

MD5 authentication is **officially deprecated in PostgreSQL 18**. It remains supported for backward compatibility, but you should migrate all roles to SCRAM-SHA-256 as soon as possible. Future PostgreSQL versions may remove MD5 entirely.

```conf
# Only use if legacy clients truly cannot support SCRAM
host    all    legacy_app    192.168.1.0/24    md5
```

### Password Policies

PostgreSQL does not have built-in password complexity rules, but you can use the `passwordcheck` extension:

```sql
-- In postgresql.conf:
-- shared_preload_libraries = 'passwordcheck'
-- Rejects weak passwords at CREATE/ALTER ROLE time
```

### Password Expiration

```sql
ALTER ROLE app_user VALID UNTIL '2027-12-31';

SELECT rolname, rolvaliduntil
FROM pg_roles
WHERE rolvaliduntil IS NOT NULL;
```

## Common Pitfalls
1. **Forgetting to re-set passwords after changing encryption** — Changing `password_encryption` to `scram-sha-256` only affects new passwords. Existing MD5-hashed passwords must be explicitly re-set with `ALTER ROLE ... PASSWORD '...'`.
2. **Using MD5 for new deployments** — MD5 is deprecated in PostgreSQL 18. Always start new projects with SCRAM-SHA-256.

## Best Practices
1. **Set `password_encryption = 'scram-sha-256'` globally** — This ensures every new or changed password is stored with the strongest available hash.
2. **Use VALID UNTIL for service accounts** — Forcing periodic password rotation catches forgotten credentials and limits the window of exposure if a password leaks.

## Summary
- SCRAM-SHA-256 is the recommended and most secure password authentication method.
- MD5 is officially deprecated in PostgreSQL 18 — plan your migration now.
- Use `passwordcheck` for complexity enforcement and `VALID UNTIL` for expiration.

## Code Examples

**Configuring SCRAM-SHA-256 encryption and verifying password storage format**

```sql
-- Set encryption to SCRAM and create a user
ALTER SYSTEM SET password_encryption = 'scram-sha-256';
SELECT pg_reload_conf();

CREATE ROLE app_user WITH LOGIN PASSWORD 'secure_pass';

-- Verify the password hash format
SELECT rolname, left(rolpassword, 14) AS hash_prefix
FROM pg_authid WHERE rolname = 'app_user';
-- Should show: SCRAM-SHA-256$
```


## Resources

- [Password Authentication](https://www.postgresql.org/docs/18/auth-password.html) — Password authentication methods reference

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*