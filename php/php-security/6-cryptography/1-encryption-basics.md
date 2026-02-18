---
source_course: "php-security"
source_lesson: "php-security-encryption-basics"
---

# Encryption Fundamentals

Encryption protects data confidentiality. Understanding when and how to use it is crucial.

## Hashing vs Encryption

| Hashing | Encryption |
|---------|------------|
| One-way (irreversible) | Two-way (reversible) |
| Same input = same output | Different each time (with IV) |
| For passwords, integrity | For data you need back |
| password_hash(), hash() | openssl_encrypt() |

## Symmetric Encryption (AES)

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

## Generating Secure Keys

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

## Secure Random Generation

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

## When to Use Encryption

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