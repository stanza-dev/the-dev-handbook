---
source_course: "postgresql-security"
source_lesson: "postgresql-security-encryption"
---

# Data Encryption

## Introduction
While SSL protects data in transit, data at rest needs separate protection. PostgreSQL's pgcrypto extension provides hashing, symmetric encryption, and asymmetric encryption directly in the database. Understanding when and how to use each technique is critical for protecting sensitive data stored on disk.

## Key Concepts
- **Hashing**: One-way transformation used for password verification (bcrypt via `crypt()`) and integrity checking (SHA-256).
- **Symmetric Encryption**: Same key for encryption and decryption, provided by `pgp_sym_encrypt()` and `pgp_sym_decrypt()`.
- **Key Management**: The practice of securely storing and rotating encryption keys — never in the database alongside the encrypted data.

## Real World Context
A healthcare application stores patient notes that must be encrypted at rest for HIPAA compliance. Using `pgp_sym_encrypt()` with a key from an external secrets manager (like AWS Secrets Manager or HashiCorp Vault), you can encrypt sensitive columns while keeping non-sensitive metadata queryable. The tradeoff: encrypted columns cannot be indexed or searched.

## Deep Dive

### pgcrypto Extension

```sql
CREATE EXTENSION pgcrypto;
```

### Hashing

```sql
-- bcrypt hash for passwords
SELECT crypt('user_password', gen_salt('bf'));

-- Verify password
SELECT crypt('user_password', stored_hash) = stored_hash;

-- SHA-256 hash for data integrity
SELECT digest('sensitive data', 'sha256');
```

### Symmetric Encryption

```sql
-- Encrypt
SELECT pgp_sym_encrypt('secret message', 'encryption_key');

-- Decrypt
SELECT pgp_sym_decrypt(encrypted_column::bytea, 'encryption_key');
```

### Storing Encrypted Data

```sql
CREATE TABLE secure_notes (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    encrypted_content BYTEA,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO secure_notes (user_id, encrypted_content)
VALUES (1, pgp_sym_encrypt('This is secret', current_setting('app.encryption_key')));

SELECT id,
    pgp_sym_decrypt(encrypted_content, current_setting('app.encryption_key')) AS content
FROM secure_notes WHERE user_id = 1;
```

### Key Management Best Practices

1. **Never store keys in the database**
2. Use environment variables or a secrets manager
3. Rotate keys periodically
4. Use different keys for different data classifications

```sql
SET app.encryption_key = 'key_from_env_or_vault';
```

### Transparent Data Encryption (TDE)

PostgreSQL does not have built-in TDE, but options include filesystem encryption (LUKS, BitLocker), cloud provider encryption (AWS RDS, Azure Database), and third-party extensions.

## Common Pitfalls
1. **Storing the encryption key in the database** — If an attacker gains database access, they get both the encrypted data and the key. Always store keys externally.
2. **Trying to search encrypted columns** — Encrypted data cannot be indexed or pattern-matched. Design your schema so searchable fields remain unencrypted.

## Best Practices
1. **Use pgp_sym_encrypt/decrypt for application-level encryption** — This provides per-column encryption with standard PGP-compatible algorithms.
2. **Separate encryption keys by data classification** — Use different keys for different sensitivity levels so a compromised key has limited blast radius.

## Summary
- pgcrypto provides hashing, symmetric, and asymmetric encryption within PostgreSQL.
- Never store encryption keys in the database — use external secrets management.
- Encrypted columns cannot be searched or indexed — plan your schema accordingly.

## Code Examples

**Encrypting and decrypting sensitive data with pgcrypto symmetric encryption**

```sql
-- Encrypt and decrypt with pgcrypto
CREATE EXTENSION pgcrypto;

-- Store encrypted data
INSERT INTO secure_notes (user_id, encrypted_content)
VALUES (1, pgp_sym_encrypt(
    'Patient record: confidential',
    current_setting('app.encryption_key')
));

-- Retrieve decrypted data
SELECT pgp_sym_decrypt(
    encrypted_content,
    current_setting('app.encryption_key')
) AS content
FROM secure_notes WHERE user_id = 1;
```


## Resources

- [pgcrypto](https://www.postgresql.org/docs/18/pgcrypto.html) — Cryptographic functions

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*