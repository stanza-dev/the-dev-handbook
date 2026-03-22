---
source_course: "postgresql-security"
source_lesson: "postgresql-security-client-certs"
---

# Client Certificate Authentication

## Introduction
Client certificate authentication (mutual TLS or mTLS) provides the strongest form of authentication in PostgreSQL. Instead of passwords, clients present a cryptographic certificate signed by a trusted CA. This eliminates password-related risks entirely — no passwords to steal, phish, or brute-force.

## Key Concepts
- **Mutual TLS (mTLS)**: Both the server and client verify each other's certificates, providing two-way authentication.
- **Client Certificate**: A certificate file (client.crt) and private key (client.key) that prove the client's identity.
- **Certificate Authority (CA)**: The trusted entity that signs both server and client certificates.

## Real World Context
In a microservices architecture, each service connects to PostgreSQL. Using client certificates means you do not need to distribute or rotate passwords. Each service gets its own certificate from the internal CA, and certificate revocation instantly blocks a compromised service. This is the standard approach in zero-trust network architectures.

## Deep Dive

### Setting Up the CA

```bash
# Generate CA key and certificate
openssl req -new -x509 -days 3650 -nodes \
    -out ca.crt -keyout ca.key \
    -subj "/CN=PostgreSQL Internal CA"
```

### Generating Client Certificates

```bash
# Generate client key
openssl genrsa -out client.key 2048

# Generate certificate signing request
openssl req -new -key client.key \
    -out client.csr \
    -subj "/CN=app_user"

# Sign with CA
openssl x509 -req -in client.csr \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -out client.crt -days 365

chmod 600 client.key
```

Note: The CN (Common Name) in the client certificate must match the PostgreSQL role name.

### Server Configuration

```ini
# postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'ca.crt'      # CA that signed client certs
```

```conf
# pg_hba.conf - require client certificate
hostssl  all  all  0.0.0.0/0  cert
```

### Client Connection

```bash
psql "host=db.example.com dbname=myapp \
      sslmode=verify-full \
      sslcert=client.crt \
      sslkey=client.key \
      sslrootcert=ca.crt"
```

### Mapping Certificates to Roles

By default, the certificate's CN must match the database role. You can customize this with `pg_ident.conf`:

```conf
# pg_hba.conf
hostssl  all  all  0.0.0.0/0  cert map=cert_map

# pg_ident.conf
cert_map  /^(.*)@example\.com$   \1
```

## Common Pitfalls
1. **Not setting proper file permissions on client.key** — The private key must be readable only by the owner (chmod 600). PostgreSQL and libpq will refuse keys with overly permissive permissions.
2. **CN mismatch** — The certificate's CN must match the PostgreSQL role name (unless you configure a pg_ident.conf mapping).

## Best Practices
1. **Use short-lived certificates** — Issue client certificates with short validity periods (30-90 days) and automate renewal to limit exposure from compromised certificates.
2. **Implement certificate revocation** — Configure CRL (Certificate Revocation List) or OCSP checking so you can immediately block compromised certificates.

## Summary
- Client certificate authentication eliminates passwords entirely, using cryptographic proof of identity.
- The certificate CN must match the PostgreSQL role name (or use pg_ident.conf mapping).
- Use short-lived certificates and implement revocation for maximum security.

## Code Examples

**Configuring client certificate auth and inspecting connected client certificate details**

```sql
-- pg_hba.conf entry for client certificate auth
-- hostssl  all  all  0.0.0.0/0  cert

-- Verify a client's certificate details from SQL
SELECT pid, usename, ssl, client_dn, client_serial
FROM pg_stat_ssl
JOIN pg_stat_activity USING (pid)
WHERE ssl = true;
```


## Resources

- [SSL Certificate Authentication](https://www.postgresql.org/docs/18/auth-cert.html) — Certificate-based authentication reference

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*