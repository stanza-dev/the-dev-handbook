---
source_course: "postgresql-security"
source_lesson: "postgresql-security-ssl-config"
---

# Configuring SSL/TLS

Encrypt connections between clients and the PostgreSQL server.

## Why SSL/TLS?

- **Encryption**: Data cannot be read in transit
- **Authentication**: Verify server identity
- **Integrity**: Detect tampering

## Enabling SSL on Server

```ini
# postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'ca.crt'  # For client certificates
```

## Certificate Setup

```bash
# Generate self-signed certificate (testing only)
openssl req -new -x509 -days 365 -nodes \
    -out server.crt \
    -keyout server.key \
    -subj "/CN=postgresql-server"

# Set permissions
chmod 600 server.key
chown postgres:postgres server.key server.crt
```

## Requiring SSL in pg_hba.conf

```conf
# Require SSL for remote connections
hostssl  all  all  0.0.0.0/0  scram-sha-256

# Allow non-SSL only from localhost
host     all  all  127.0.0.1/32  scram-sha-256

# Reject non-SSL remote connections
hostnossl all all  0.0.0.0/0  reject
```

## Client SSL Options

```bash
# Connection string with SSL
psql "host=db.example.com dbname=myapp sslmode=require"

# Or environment variable
export PGSSLMODE=require
```

### sslmode Options

| Mode | Encryption | Server Cert Check |
|------|------------|-------------------|
| `disable` | No | No |
| `allow` | If server requires | No |
| `prefer` | If available | No |
| `require` | **Yes** | No |
| `verify-ca` | **Yes** | CA only |
| `verify-full` | **Yes** | CA + hostname |

**Recommended for production:** `verify-full`

```bash
psql "host=db.example.com sslmode=verify-full sslrootcert=/path/to/ca.crt"
```

## Checking SSL Status

```sql
-- Check current connection
SELECT ssl_is_used();

-- View SSL info for all connections
SELECT 
    pid,
    usename,
    ssl,
    client_addr
FROM pg_stat_ssl 
JOIN pg_stat_activity USING (pid);
```

📖 [SSL/TLS](https://www.postgresql.org/docs/18/ssl-tcp.html)

## Resources

- [Secure TCP/IP with SSL](https://www.postgresql.org/docs/18/ssl-tcp.html) — SSL configuration guide

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*