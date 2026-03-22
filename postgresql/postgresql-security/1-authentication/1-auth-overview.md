---
source_course: "postgresql-security"
source_lesson: "postgresql-security-auth-overview"
---

# Authentication Overview

## Introduction
Authentication is the first line of defense for any PostgreSQL database. It verifies **who** a user claims to be before granting any access. Understanding the authentication pipeline is critical because a misconfiguration here can expose your entire database to unauthorized access.

## Key Concepts
- **pg_hba.conf**: The host-based authentication configuration file that determines which authentication method to use for each connection.
- **Authentication Method**: The mechanism used to verify a user's identity (e.g., password, certificate, LDAP, OAuth).
- **Connection Type**: Whether the connection arrives via Unix socket (`local`), TCP/IP (`host`), or SSL-encrypted TCP/IP (`hostssl`).

## Real World Context
Every production PostgreSQL deployment must configure authentication carefully. A single `trust` rule left in pg_hba.conf on a public-facing server means anyone can connect without a password. In practice, teams use SCRAM-SHA-256 for password auth, certificate-based auth for service-to-service communication, and LDAP or OAuth for enterprise SSO.

## Deep Dive

PostgreSQL processes every incoming connection through a pipeline:

```
┌─────────────────────────────────────────────────────────┐
│                    Client Connection                     │
└────────────────────────────────┬────────────────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────┐
│              pg_hba.conf (Host-Based Access)            │
│  Determines: Which method for this user/host/database?  │
└────────────────────────────────┬────────────────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────┐
│              Authentication Method Execution             │
│  trust | password | scram-sha-256 | cert | ldap | ...   │
└────────────────────────────────┬────────────────────────┘
                                 ▼
                        ✓ Connected / ✗ Rejected
```

The following table summarizes the available authentication methods:

| Method | Security | Use Case |
|--------|----------|----------|
| `trust` | None | Development only! |
| `password` | Low | Legacy systems |
| `md5` | Medium | Deprecated in PostgreSQL 18, compatibility only |
| `scram-sha-256` | High | **Recommended for passwords** |
| `cert` | Very High | Certificate-based |
| `peer` | High | Local Unix connections |
| `ldap` | High | Enterprise directory |
| `oauth` | High | Modern authentication (PostgreSQL 18+) |

To find your pg_hba.conf location:

```sql
SHOW hba_file;
```

Common locations are `/etc/postgresql/18/main/pg_hba.conf` on Linux, `/opt/homebrew/var/postgresql@18/pg_hba.conf` on macOS, and `C:\Program Files\PostgreSQL\18\data\pg_hba.conf` on Windows.

After editing pg_hba.conf, reload the configuration:

```sql
SELECT pg_reload_conf();
```

## Common Pitfalls
1. **Leaving `trust` in production** — This allows anyone matching the rule to connect without a password. Always remove or restrict trust rules before deploying.
2. **Using `md5` when SCRAM is available** — MD5 is officially deprecated in PostgreSQL 18. Migrate to `scram-sha-256` for all password-based authentication.

## Best Practices
1. **Use `scram-sha-256` as default** — It provides salted challenge-response authentication without transmitting the password over the network.
2. **Restrict `trust` to local development** — Never allow trust authentication from any network address in production environments.

## Summary
- PostgreSQL authentication is driven by pg_hba.conf, which maps connections to authentication methods.
- SCRAM-SHA-256 is the recommended password method; MD5 is deprecated in PostgreSQL 18.
- Always reload configuration after changes and test with a non-superuser account.

## Code Examples

**Finding and reloading pg_hba.conf, and inspecting active rules**

```sql
-- Find your pg_hba.conf location
SHOW hba_file;

-- Reload configuration after editing pg_hba.conf
SELECT pg_reload_conf();

-- Check current connection's auth method
SELECT * FROM pg_hba_file_rules
WHERE type = 'host'
ORDER BY line_number;
```


## Resources

- [Client Authentication](https://www.postgresql.org/docs/18/client-authentication.html) — Complete authentication guide
- [Authentication Methods](https://www.postgresql.org/docs/18/auth-methods.html) — All supported authentication methods

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*