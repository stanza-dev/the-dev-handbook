---
source_course: "postgresql-security"
source_lesson: "postgresql-security-encryption"
---

# Data Encryption

Protect sensitive data at rest and in application.

## pgcrypto Extension

```sql
CREATE EXTENSION pgcrypto;
```

### Hashing

```sql
-- Hash a password (for verification, not storage)
SELECT crypt('user_password', gen_salt('bf'));
-- Returns: $2a$06$r6LmB...

-- Verify password
SELECT crypt('user_password', stored_hash) = stored_hash;

-- SHA-256 hash
SELECT digest('sensitive data', 'sha256');
```

### Symmetric Encryption

```sql
-- Encrypt data
SELECT pgp_sym_encrypt('secret message', 'encryption_key');

-- Decrypt data
SELECT pgp_sym_decrypt(
    encrypted_column::bytea, 
    'encryption_key'
);
```

### Storing Encrypted Data

```sql
CREATE TABLE secure_notes (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    encrypted_content BYTEA,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Insert encrypted data
INSERT INTO secure_notes (user_id, encrypted_content)
VALUES (
    1,
    pgp_sym_encrypt('This is secret', current_setting('app.encryption_key'))
);

-- Read decrypted data
SELECT 
    id,
    pgp_sym_decrypt(encrypted_content, current_setting('app.encryption_key')) AS content
FROM secure_notes
WHERE user_id = 1;
```

## Key Management Best Practices

1. **Never store keys in the database**
2. Use environment variables or secrets manager
3. Rotate keys periodically
4. Use different keys for different data classifications

```sql
-- Set key from environment (application should do this)
SET app.encryption_key = 'key_from_env_or_vault';
```

## Transparent Data Encryption (TDE)

PostgreSQL doesn't have built-in TDE, but options include:

1. **Filesystem encryption**: LUKS, BitLocker
2. **Cloud provider encryption**: AWS RDS, Azure Database
3. **Third-party extensions**: Some enterprise solutions

## Column-Level Encryption Considerations

**Pros:**
- Fine-grained control
- Only sensitive data encrypted
- Can search unencrypted columns

**Cons:**
- Cannot search/index encrypted columns
- Key management complexity
- Performance overhead

📖 [pgcrypto](https://www.postgresql.org/docs/18/pgcrypto.html)

## Resources

- [pgcrypto](https://www.postgresql.org/docs/18/pgcrypto.html) — Cryptographic functions

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*