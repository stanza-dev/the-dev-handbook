---
source_course: "postgresql-security"
source_lesson: "postgresql-security-password-auth"
---

# Password Authentication

Securely authenticate users with passwords using SCRAM-SHA-256.

## SCRAM-SHA-256 (Recommended)

**S**alted **C**hallenge **R**esponse **A**uthentication **M**echanism:

- Password never sent over network
- Server doesn't store plain password
- Protection against replay attacks
- Resistant to brute-force attacks

```conf
# pg_hba.conf
host    all    all    0.0.0.0/0    scram-sha-256
```

## Setting Password Encryption

```sql
-- Check current setting
SHOW password_encryption;

-- Set to SCRAM (recommended)
ALTER SYSTEM SET password_encryption = 'scram-sha-256';
SELECT pg_reload_conf();
```

## Creating Users with Passwords

```sql
-- Create user with password
CREATE ROLE app_user WITH LOGIN PASSWORD 'secure_password_here';

-- Update existing user's password
ALTER ROLE app_user WITH PASSWORD 'new_secure_password';

-- Check password encryption
SELECT rolname, rolpassword 
FROM pg_authid 
WHERE rolname = 'app_user';
-- SCRAM passwords start with 'SCRAM-SHA-256$'
```

## MD5 (Legacy)

Still supported for compatibility but **not recommended**:

```conf
# Only use if clients don't support SCRAM
host    all    legacy_app    192.168.1.0/24    md5
```

## Password Policies

PostgreSQL doesn't have built-in password policies. Options:

**1. Use passwordcheck extension:**
```sql
-- postgresql.conf
shared_preload_libraries = 'passwordcheck'

-- Rejects weak passwords at CREATE/ALTER ROLE
```

**2. Application-level validation:**
```plpgsql
CREATE OR REPLACE FUNCTION validate_password()
RETURNS event_trigger AS $$
DECLARE
    r RECORD;
BEGIN
    FOR r IN SELECT * FROM pg_event_trigger_ddl_commands() LOOP
        IF r.command_tag = 'CREATE ROLE' OR r.command_tag = 'ALTER ROLE' THEN
            -- Custom validation logic
        END IF;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

## Password Expiration

```sql
-- Set password valid until date
ALTER ROLE app_user VALID UNTIL '2025-12-31';

-- Check expiration
SELECT rolname, rolvaliduntil 
FROM pg_roles 
WHERE rolvaliduntil IS NOT NULL;
```

📖 [Password Authentication](https://www.postgresql.org/docs/18/auth-password.html)

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*