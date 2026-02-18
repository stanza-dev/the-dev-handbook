---
source_course: "postgresql-security"
source_lesson: "postgresql-security-auth-overview"
---

# Authentication Overview

Authentication verifies **who** a user claims to be. PostgreSQL supports multiple authentication methods to fit different security requirements.

## The Authentication Pipeline

```
┌─────────────────────────────────────────────────────────┐
│                    Client Connection                     │
└────────────────────────────┬────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────┐
│              pg_hba.conf (Host-Based Access)            │
│  Determines: Which method for this user/host/database?  │
└────────────────────────────┬────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────┐
│              Authentication Method Execution             │
│  trust | password | scram-sha-256 | cert | ldap | ...   │
└────────────────────────────┬────────────────────────────┘
                             ▼
                    ✓ Connected / ✗ Rejected
```

## Available Authentication Methods

| Method | Security | Use Case |
|--------|----------|----------|
| `trust` | None | Development only! |
| `password` | Low | Legacy systems |
| `md5` | Medium | Compatibility |
| `scram-sha-256` | High | **Recommended for passwords** |
| `cert` | Very High | Certificate-based |
| `peer` | High | Local Unix connections |
| `ldap` | High | Enterprise directory |
| `oauth` | High | Modern authentication (PostgreSQL 18+) |

## pg_hba.conf Location

```bash
# Find your pg_hba.conf
psql -c "SHOW hba_file;"

# Common locations:
# Linux: /etc/postgresql/18/main/pg_hba.conf
# macOS (Homebrew): /opt/homebrew/var/postgresql@18/pg_hba.conf
# Windows: C:\Program Files\PostgreSQL\18\data\pg_hba.conf
```

## Reloading Configuration

After editing pg_hba.conf:

```sql
-- From SQL
SELECT pg_reload_conf();

-- From command line
pg_ctl reload -D /path/to/data
```

📖 [Authentication Methods](https://www.postgresql.org/docs/18/auth-methods.html)

## Resources

- [Client Authentication](https://www.postgresql.org/docs/18/client-authentication.html) — Complete authentication guide

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*