---
source_course: "postgresql-security"
source_lesson: "postgresql-security-pg-hba-conf"
---

# Configuring pg_hba.conf

## Introduction
The pg_hba.conf file is the gatekeeper for every PostgreSQL connection. Mastering its syntax means you can precisely control who connects, from where, and how they authenticate. A misconfigured pg_hba.conf is one of the most common sources of both security breaches and frustrating connection failures.

## Key Concepts
- **Rule Ordering**: PostgreSQL uses the **first matching rule** from top to bottom. Order matters critically.
- **Connection Type**: `local` (Unix socket), `host` (TCP/IP), `hostssl` (SSL required), `hostnossl` (no SSL), `hostgssenc` (GSSAPI encryption).
- **Address Pattern**: CIDR notation for IP ranges, hostnames with DNS, or `all` for any address.

## Real World Context
Imagine deploying a new application server that cannot connect to the database. The issue is almost always in pg_hba.conf: either the server's IP is not covered by any rule, or a broader reject rule appears before the intended allow rule. Understanding rule ordering prevents hours of debugging.

## Deep Dive

Each line in pg_hba.conf follows this format:

```conf
# TYPE  DATABASE  USER  ADDRESS         METHOD  [OPTIONS]
local   all       all                   peer
host    all       all   127.0.0.1/32    scram-sha-256
host    all       all   ::1/128         scram-sha-256
```

### Connection Types

| Type | Description |
|------|-------------|
| `local` | Unix domain socket connections |
| `host` | TCP/IP (SSL or not) |
| `hostssl` | TCP/IP with SSL required |
| `hostnossl` | TCP/IP without SSL |
| `hostgssenc` | TCP/IP with GSSAPI encryption |

### Database and User Fields

You can specify individual databases, comma-separated lists, `all`, or use the `+` prefix for role membership:

```conf
host    myapp       all        192.168.1.0/24    scram-sha-256
host    db1,db2     all        192.168.1.0/24    scram-sha-256
host    all         +admins    192.168.1.0/24    scram-sha-256
```

### Address Patterns

```conf
host    all    all    192.168.1.100/32    scram-sha-256   # Single IP
host    all    all    192.168.1.0/24      scram-sha-256   # Subnet
host    all    all    0.0.0.0/0           scram-sha-256   # All IPv4
host    all    all    .example.com        scram-sha-256   # Hostname
```

Here is a complete production example:

```conf
# TYPE  DATABASE    USER        ADDRESS           METHOD
local   all         postgres                      peer
local   all         all                           scram-sha-256
host    all         all         127.0.0.1/32      scram-sha-256
host    all         all         ::1/128           scram-sha-256
hostssl myapp       app_user    10.0.1.0/24       scram-sha-256
host    all         admin       203.0.113.0/24    scram-sha-256
host    all         all         0.0.0.0/0         reject
```

Remember: PostgreSQL reads rules top to bottom and stops at the first match.

## Common Pitfalls
1. **Placing a broad reject rule too early** — A `host all all 0.0.0.0/0 reject` at the top blocks every remote connection, including legitimate ones. Always put reject rules last.
2. **Forgetting to reload after edits** — Changes to pg_hba.conf do not take effect until you run `SELECT pg_reload_conf()` or restart the server.

## Best Practices
1. **Order rules from most specific to least specific** — Put individual user/IP rules before subnet rules, and subnet rules before catch-all rules.
2. **End with an explicit reject rule** — Add `host all all 0.0.0.0/0 reject` as the last line to deny any connection that was not explicitly allowed.

## Summary
- pg_hba.conf uses first-match semantics, so rule order is critical.
- Use `hostssl` to enforce encrypted connections for remote access.
- Always end with a reject-all rule as a safety net.

## Code Examples

**Inspecting active pg_hba.conf rules and reloading configuration**

```sql
-- View parsed pg_hba.conf rules from SQL
SELECT line_number, type, database, user_name, address, auth_method
FROM pg_hba_file_rules
ORDER BY line_number;

-- Reload after changes
SELECT pg_reload_conf();
```


## Resources

- [pg_hba.conf Documentation](https://www.postgresql.org/docs/18/auth-pg-hba-conf.html) — Complete pg_hba.conf reference

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*