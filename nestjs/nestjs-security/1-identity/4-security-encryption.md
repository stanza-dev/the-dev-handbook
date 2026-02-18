---
source_course: "nestjs-security"
source_lesson: "nestjs-security-encryption"
---

# Encryption & Hashing

## Introduction

Security isn't just about authentication—it's about protecting sensitive data. Passwords must be hashed, not encrypted. Sensitive data in transit and at rest needs encryption. Understanding the difference is critical for building secure applications.

## Key Concepts

- **Hashing**: One-way transformation (cannot be reversed)—used for passwords
- **Encryption**: Two-way transformation (can be decrypted)—used for sensitive data
- **Salt**: Random data added to input before hashing to prevent rainbow table attacks
- **bcrypt**: Industry-standard password hashing algorithm

## Real World Context

- **Passwords**: Always hash with bcrypt, never encrypt or store plaintext
- **API Keys**: Hash for storage, display only on creation
- **PII**: Encrypt social security numbers, credit cards at rest
- **Tokens**: Use cryptographically secure random generation

## Deep Dive

### Password Hashing with bcrypt

```bash
npm install bcrypt
npm install -D @types/bcrypt
```

```typescript
import * as bcrypt from 'bcrypt';

@Injectable()
export class UsersService {
  private readonly saltRounds = 10;

  async createUser(email: string, password: string) {
    const hashedPassword = await bcrypt.hash(password, this.saltRounds);
    
    return this.usersRepository.create({
      email,
      password: hashedPassword,
    });
  }

  async validatePassword(plainPassword: string, hashedPassword: string): Promise<boolean> {
    return bcrypt.compare(plainPassword, hashedPassword);
  }
}
```

### Why bcrypt?

- **Adaptive**: The cost factor can be increased as hardware improves
- **Salted**: Automatically includes a salt in the hash
- **Slow by design**: Makes brute-force attacks impractical

### Encryption for Sensitive Data

```typescript
import * as crypto from 'crypto';

@Injectable()
export class EncryptionService {
  private readonly algorithm = 'aes-256-gcm';
  private readonly key = Buffer.from(process.env.ENCRYPTION_KEY, 'hex');

  encrypt(text: string): { encrypted: string; iv: string; tag: string } {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      tag: cipher.getAuthTag().toString('hex'),
    };
  }

  decrypt(encrypted: string, iv: string, tag: string): string {
    const decipher = crypto.createDecipheriv(
      this.algorithm,
      this.key,
      Buffer.from(iv, 'hex'),
    );
    decipher.setAuthTag(Buffer.from(tag, 'hex'));
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}
```

### Secure Random Token Generation

```typescript
import * as crypto from 'crypto';

function generateSecureToken(length: number = 32): string {
  return crypto.randomBytes(length).toString('hex');
}

// For API keys
const apiKey = generateSecureToken(32); // 64 hex characters

// For password reset tokens
const resetToken = generateSecureToken(16); // 32 hex characters
```

## Common Pitfalls

1. **Encrypting passwords**: Passwords should be hashed, not encrypted. If you can decrypt it, attackers can too.
2. **Using MD5 or SHA1 for passwords**: These are too fast. Use bcrypt, scrypt, or Argon2.
3. **Hardcoding encryption keys**: Store keys in environment variables or secret managers.

## Best Practices

- Use bcrypt with cost factor 10+ for passwords
- Use AES-256-GCM for encrypting sensitive data
- Generate unique IVs for each encryption operation
- Store encryption keys in secret managers (AWS Secrets Manager, HashiCorp Vault)
- Rotate encryption keys periodically

## Summary

Hash passwords with bcrypt (one-way, salted, slow). Encrypt sensitive data with AES-256-GCM. Generate secure tokens with crypto.randomBytes. Never roll your own cryptography—use established libraries.

## Resources

- [Encryption & Hashing](https://docs.nestjs.com/security/encryption-and-hashing) — Official security guide

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*