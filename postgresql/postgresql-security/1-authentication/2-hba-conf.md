---
source_course: "postgresql-security"
source_lesson: "postgresql-security-pg-hba-conf"
---

# Configuring pg_hba.conf

The pg_hba.conf file controls client authentication rules.

## File Format

```conf
# TYPE  DATABASE  USER  ADDRESS         METHOD  [OPTIONS]
local   all       all                   peer
host    all       all   127.0.0.1/32    scram-sha-256
host    all       all   ::1/128         scram-sha-256
```

## Connection Types

| Type | Description |
|------|-------------|
| `local` | Unix domain socket connections |
| `host` | TCP/IP (SSL or not) |
| `hostssl` | TCP/IP with SSL required |
| `hostnossl` | TCP/IP without SSL |
| `hostgssenc` | TCP/IP with GSSAPI encryption |

## Database Field

```conf
# Specific database
host    myapp       all   192.168.1.0/24    scram-sha-256

# Multiple databases
host    db1,db2     all   192.168.1.0/24    scram-sha-256

# All databases
host    all         all   192.168.1.0/24    scram-sha-256

# All except some
host    all,@users  all   192.168.1.0/24    scram-sha-256
```

## User Field

```conf
# Specific user
host    all         admin    192.168.1.0/24    scram-sha-256

# Multiple users
host    all         admin,dba 192.168.1.0/24   scram-sha-256

# Role membership (+prefix)
host    all         +admins   192.168.1.0/24   scram-sha-256

# All users
host    all         all       192.168.1.0/24   scram-sha-256
```

## Address Patterns

```conf
# Single IP (CIDR notation)
host    all    all    192.168.1.100/32    scram-sha-256

# Subnet
host    all    all    192.168.1.0/24      scram-sha-256

# All IPv4
host    all    all    0.0.0.0/0           scram-sha-256

# All IPv6
host    all    all    ::0/0               scram-sha-256

# Hostname (with DNS)
host    all    all    .example.com        scram-sha-256
```

## Complete Example

```conf
# TYPE  DATABASE    USER        ADDRESS           METHOD

# Local connections via Unix socket
local   all         postgres                      peer
local   all         all                           scram-sha-256

# Localhost IPv4 and IPv6
host    all         all         127.0.0.1/32      scram-sha-256
host    all         all         ::1/128           scram-sha-256

# Application servers (require SSL)
hostssl  myapp      app_user    10.0.1.0/24       scram-sha-256

# Admin access from office (specific users)
host    all         admin       203.0.113.0/24    scram-sha-256

# Reject everything else
host    all         all         0.0.0.0/0         reject
```

## Order Matters!

PostgreSQL uses the **first matching rule**:

```conf
# This user gets md5 (matches first)
host    all    admin    192.168.1.0/24    md5
host    all    all      192.168.1.0/24    scram-sha-256
```

📖 [The pg_hba.conf File](https://www.postgresql.org/docs/18/auth-pg-hba-conf.html)

## Resources

- [pg_hba.conf Documentation](https://www.postgresql.org/docs/18/auth-pg-hba-conf.html) — Complete pg_hba.conf reference

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*