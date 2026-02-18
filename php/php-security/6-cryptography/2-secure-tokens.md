---
source_course: "php-security"
source_lesson: "php-security-secure-tokens"
---

# Generating Secure Tokens

Secure random tokens are essential for session IDs, CSRF tokens, password reset links, and API keys.

## Cryptographically Secure Randomness

```php
<?php
// PHP 7+ - Always use random_bytes() for security
$bytes = random_bytes(32);  // 32 bytes = 256 bits
$token = bin2hex($bytes);   // 64 character hex string

// For URL-safe tokens
$urlSafeToken = rtrim(strtr(base64_encode(random_bytes(32)), '+/', '-_'), '=');
```

## Token Generation Patterns

```php
<?php
class TokenGenerator
{
    public function generateHex(int $bytes = 32): string
    {
        return bin2hex(random_bytes($bytes));
    }
    
    public function generateUrlSafe(int $bytes = 32): string
    {
        // URL-safe Base64 (no +, /, or =)
        return rtrim(strtr(base64_encode(random_bytes($bytes)), '+/', '-_'), '=');
    }
    
    public function generateAlphanumeric(int $length = 32): string
    {
        $chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
        $result = '';
        
        for ($i = 0; $i < $length; $i++) {
            $result .= $chars[random_int(0, 61)];
        }
        
        return $result;
    }
    
    public function generateUuid(): string
    {
        $data = random_bytes(16);
        
        // Set version to 4 (random)
        $data[6] = chr(ord($data[6]) & 0x0f | 0x40);
        // Set bits 6-7 to 10
        $data[8] = chr(ord($data[8]) & 0x3f | 0x80);
        
        return vsprintf('%s%s-%s-%s-%s-%s%s%s', str_split(bin2hex($data), 4));
    }
}
```

## Password Reset Tokens

```php
<?php
class PasswordReset
{
    private const TOKEN_EXPIRY = 3600;  // 1 hour
    
    public function createToken(int $userId): string
    {
        // Generate token
        $selector = bin2hex(random_bytes(16));  // For lookup
        $validator = bin2hex(random_bytes(32)); // For verification
        
        // Store hashed validator (selector is stored plain for lookup)
        $this->db->insert('password_resets', [
            'user_id' => $userId,
            'selector' => $selector,
            'hashed_validator' => hash('sha256', $validator),
            'expires_at' => date('Y-m-d H:i:s', time() + self::TOKEN_EXPIRY),
        ]);
        
        // Return combined token
        return $selector . ':' . $validator;
    }
    
    public function validateToken(string $token): ?int
    {
        [$selector, $validator] = explode(':', $token, 2);
        
        $record = $this->db->findOne('password_resets', [
            'selector' => $selector,
        ]);
        
        if (!$record) {
            return null;
        }
        
        // Check expiry
        if (strtotime($record['expires_at']) < time()) {
            $this->db->delete('password_resets', ['id' => $record['id']]);
            return null;
        }
        
        // Verify validator (constant-time comparison)
        if (!hash_equals($record['hashed_validator'], hash('sha256', $validator))) {
            return null;
        }
        
        // Delete used token
        $this->db->delete('password_resets', ['id' => $record['id']]);
        
        return $record['user_id'];
    }
}
```

## API Key Generation

```php
<?php
class ApiKeyManager
{
    private const PREFIX = 'sk_';  // For identification
    
    public function generate(): array
    {
        // Visible part (for identification)
        $keyId = bin2hex(random_bytes(8));
        
        // Secret part
        $secret = bin2hex(random_bytes(32));
        
        // Full key shown once to user
        $fullKey = self::PREFIX . $keyId . '_' . $secret;
        
        // Store only hash of secret
        return [
            'full_key' => $fullKey,  // Show once, never store
            'key_id' => $keyId,      // Store for lookup
            'key_hash' => hash('sha256', $secret),  // Store for verification
        ];
    }
    
    public function verify(string $apiKey): ?array
    {
        // Parse key
        if (!str_starts_with($apiKey, self::PREFIX)) {
            return null;
        }
        
        $parts = explode('_', substr($apiKey, strlen(self::PREFIX)), 2);
        if (count($parts) !== 2) {
            return null;
        }
        
        [$keyId, $secret] = $parts;
        
        // Lookup by key_id
        $record = $this->db->findOne('api_keys', ['key_id' => $keyId]);
        
        if (!$record || !$record['active']) {
            return null;
        }
        
        // Verify secret
        if (!hash_equals($record['key_hash'], hash('sha256', $secret))) {
            return null;
        }
        
        return $record;
    }
}
```

## Resources

- [random_bytes()](https://www.php.net/manual/en/function.random-bytes.php) — PHP cryptographically secure random bytes

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*