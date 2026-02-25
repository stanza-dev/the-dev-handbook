---
source_course: "php-security"
source_lesson: "php-security-encryption-basics"
---

# Encryption Fundamentals

## Introduction
Encryption protects data confidentiality. Understanding when and how to use it is crucial.

## Key Concepts
- **Symmetric Encryption**: Same key for encryption and decryption (AES-256-GCM is the recommended algorithm).
- **random_bytes()**: PHP's cryptographically secure random generator — the only safe choice for keys, IVs, and tokens.
- **Authenticated Encryption**: AES-GCM provides both confidentiality and integrity, detecting tampering.
- **Key Management**: Storing encryption keys separately from encrypted data, ideally in environment variables or a secrets manager.

## Real World Context
The choice between hashing and encryption depends on whether you need to retrieve the original data. Passwords should be hashed (one-way), while credit card numbers need encryption (reversible). Using the wrong approach (e.g., encrypting passwords or hashing credit cards) creates fundamental security failures.

## Deep Dive
### Intro

Encryption protects data confidentiality. Understanding when and how to use it is crucial.

### Hashing vs encryption

| Hashing | Encryption |
|---------|------------|
| One-way (irreversible) | Two-way (reversible) |
| Same input = same output | Different each time (with IV) |
| For passwords, integrity | For data you need back |
| password_hash(), hash() | openssl_encrypt() |

### Symmetric encryption (aes)

```php
<?php
// Same key encrypts and decrypts
function encrypt(string $plaintext, string $key): string {
    $iv = random_bytes(16);  // Initialization vector
    
    $ciphertext = openssl_encrypt(
        $plaintext,
        'AES-256-GCM',
        $key,
        OPENSSL_RAW_DATA,
        $iv,
        $tag  // Authentication tag
    );
    
    // Combine IV + tag + ciphertext
    return base64_encode($iv . $tag . $ciphertext);
}

function decrypt(string $encrypted, string $key): string|false {
    $data = base64_decode($encrypted);
    
    $iv = substr($data, 0, 16);
    $tag = substr($data, 16, 16);
    $ciphertext = substr($data, 32);
    
    return openssl_decrypt(
        $ciphertext,
        'AES-256-GCM',
        $key,
        OPENSSL_RAW_DATA,
        $iv,
        $tag
    );
}
```

### Generating secure keys

```php
<?php
// For AES-256, need 32 bytes
$key = random_bytes(32);

// Store key safely (not in code!)
// Use environment variables or key management service
$key = $_ENV['ENCRYPTION_KEY'];

// Or derive from password
$key = hash('sha256', $password, true);  // 32 bytes
```

### Secure random generation

```php
<?php
// Use random_bytes() for cryptographic randomness
$token = bin2hex(random_bytes(32));  // 64 hex characters
$apiKey = base64_encode(random_bytes(32));  // Base64 string

// NEVER use for security:
rand();
mt_rand();
uniqid();
array_rand();
```

### When to use encryption

```php
<?php
// Encrypt: Data you need to retrieve
// - Credit card numbers
// - Personal documents
// - API keys (stored)
// - Sensitive user data

// Hash: Data you only need to verify
// - Passwords
// - File integrity
// - Digital signatures
```

## Common Pitfalls
1. **Reusing initialization vectors (IVs)** — Using the same IV with the same key allows attackers to derive relationships between plaintexts. Always generate a fresh IV with `random_bytes()` for each encryption.
2. **Using ECB mode** — ECB encrypts identical blocks to identical ciphertext, revealing patterns. Always use authenticated modes like GCM or CBC with HMAC.

## Best Practices
1. **Use AES-256-GCM for symmetric encryption** — It provides both confidentiality and integrity verification in a single operation.
2. **Store keys outside your codebase** — Use environment variables, AWS KMS, or HashiCorp Vault. Never hardcode encryption keys in source code.

## Summary
- Use `random_bytes()` for generating cryptographically secure keys, IVs, and tokens.
- AES-256-GCM is the recommended symmetric encryption algorithm, providing authenticated encryption.
- Store encryption keys separately from encrypted data using environment variables or a secrets manager.

## Code Examples

**Production-ready encryption service with AES-256-GCM**

```php
<?php
declare(strict_types=1);

// Secure encryption service
class EncryptionService {
    private const CIPHER = 'AES-256-GCM';
    private const IV_LENGTH = 16;
    private const TAG_LENGTH = 16;
    
    public function __construct(
        private string $key
    ) {
        if (strlen($key) !== 32) {
            throw new InvalidArgumentException('Key must be 32 bytes for AES-256');
        }
    }
    
    public function encrypt(string $plaintext): string {
        $iv = random_bytes(self::IV_LENGTH);
        $tag = '';
        
        $ciphertext = openssl_encrypt(
            $plaintext,
            self::CIPHER,
            $this->key,
            OPENSSL_RAW_DATA,
            $iv,
            $tag,
            '',
            self::TAG_LENGTH
        );
        
        if ($ciphertext === false) {
            throw new RuntimeException('Encryption failed: ' . openssl_error_string());
        }
        
        return base64_encode($iv . $tag . $ciphertext);
    }
    
    public function decrypt(string $encrypted): string {
        $data = base64_decode($encrypted, true);
        
        if ($data === false || strlen($data) < self::IV_LENGTH + self::TAG_LENGTH + 1) {
            throw new InvalidArgumentException('Invalid encrypted data');
        }
        
        $iv = substr($data, 0, self::IV_LENGTH);
        $tag = substr($data, self::IV_LENGTH, self::TAG_LENGTH);
        $ciphertext = substr($data, self::IV_LENGTH + self::TAG_LENGTH);
        
        $plaintext = openssl_decrypt(
            $ciphertext,
            self::CIPHER,
            $this->key,
            OPENSSL_RAW_DATA,
            $iv,
            $tag
        );
        
        if ($plaintext === false) {
            throw new RuntimeException('Decryption failed - data may be tampered');
        }
        
        return $plaintext;
    }
}

// Usage
$key = hex2bin($_ENV['ENCRYPTION_KEY']);  // 64 hex chars = 32 bytes
$encryptor = new EncryptionService($key);

$encrypted = $encryptor->encrypt('Sensitive data');
$decrypted = $encryptor->decrypt($encrypted);
?>
```


## Resources

- [OpenSSL Encrypt](https://www.php.net/manual/en/function.openssl-encrypt.php) — PHP OpenSSL encryption documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*