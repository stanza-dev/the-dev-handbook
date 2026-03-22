---
source_course: "postgresql-security"
source_lesson: "postgresql-security-ssl-config"
---

# Configuring SSL/TLS

## Introduction
SSL/TLS encryption protects data in transit between clients and the PostgreSQL server. Without encryption, passwords, queries, and query results travel across the network in plaintext, visible to anyone who can intercept traffic. Configuring SSL is a fundamental security requirement for any production deployment.

## Key Concepts
- **Server Certificate**: A file (server.crt) that proves the server's identity to clients.
- **sslmode**: A client-side setting that determines the level of SSL verification (from `disable` to `verify-full`).
- **hostssl**: A pg_hba.conf connection type that requires SSL for matching connections.
- **256-bit Cancel Request Keys**: A PostgreSQL 18 improvement that strengthens the security of query cancellation requests by using 256-bit keys instead of the previous 32-bit keys, making spoofed cancel requests practically impossible.

## Real World Context
A common production setup: application servers in a private subnet connect to PostgreSQL over the internal network. Even on a "private" network, you should use SSL because internal network traffic can be intercepted through ARP spoofing, compromised switches, or misconfigured VPNs. Using `sslmode=verify-full` ensures both encryption and server identity verification.

## Deep Dive

### Enabling SSL on Server

```ini
# postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'ca.crt'
```

### Certificate Setup

```bash
# Generate self-signed certificate (testing only)
openssl req -new -x509 -days 365 -nodes \
    -out server.crt \
    -keyout server.key \
    -subj "/CN=postgresql-server"

chmod 600 server.key
chown postgres:postgres server.key server.crt
```

### Requiring SSL in pg_hba.conf

```conf
hostssl  all  all  0.0.0.0/0  scram-sha-256
host     all  all  127.0.0.1/32  scram-sha-256
hostnossl all all  0.0.0.0/0  reject
```

### Client sslmode Options

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

### Checking SSL Status

```sql
SELECT ssl_is_used();

SELECT pid, usename, ssl, client_addr
FROM pg_stat_ssl
JOIN pg_stat_activity USING (pid);
```

### PostgreSQL 18: 256-bit Cancel Request Keys

PostgreSQL 18 enhances security by using 256-bit cancel request keys instead of the previous 32-bit keys. This makes it practically impossible for an attacker to guess a valid cancel key and terminate another user's query — a subtle but important improvement for shared environments.

## Common Pitfalls
1. **Using `sslmode=require` without certificate verification** — `require` ensures encryption but does not verify the server's identity, leaving you vulnerable to man-in-the-middle attacks.
2. **Using self-signed certificates in production** — Self-signed certs cannot be properly verified. Use certificates from a trusted CA or your organization's internal CA.

## Best Practices
1. **Use `sslmode=verify-full` in production** — This provides both encryption and server identity verification.
2. **Reject non-SSL remote connections** — Add `hostnossl all all 0.0.0.0/0 reject` to pg_hba.conf to ensure all remote traffic is encrypted.

## Summary
- Configure SSL with server.crt, server.key, and optionally ca.crt in postgresql.conf.
- Use `sslmode=verify-full` on clients and `hostssl` in pg_hba.conf for maximum security.
- PostgreSQL 18 improves cancel request security with 256-bit keys.

## Code Examples

**Checking SSL status and cipher details for active connections**

```sql
-- Check SSL status for current connection
SELECT ssl_is_used();

-- View SSL details for all connections
SELECT pid, usename, ssl, version, cipher, bits
FROM pg_stat_ssl
JOIN pg_stat_activity USING (pid)
WHERE ssl = true;
```


## Resources

- [Secure TCP/IP with SSL](https://www.postgresql.org/docs/18/ssl-tcp.html) — SSL configuration guide

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*