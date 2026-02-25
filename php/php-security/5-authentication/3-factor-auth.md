---
source_course: "php-security"
source_lesson: "php-security-multi-factor-auth"
---

# Implementing Multi-Factor Authentication

## Introduction
MFA adds an additional layer of security beyond passwords. TOTP (Time-based One-Time Password) is the most common approach.

## Key Concepts
- **TOTP (Time-based One-Time Password)**: RFC 6238 standard using shared secret + time to generate 6-digit codes every 30 seconds.
- **Backup Codes**: Pre-generated one-time recovery codes, hashed with `password_hash()` and stored in the database.
- **Recovery Flow**: Secure process for users who lose their MFA device, typically using backup codes or identity verification.
- **HOTP vs TOTP**: HOTP is counter-based (RFC 4226), TOTP is time-based and more commonly used.

## Real World Context
Google reported that enabling 2FA blocks 100% of automated bot attacks, 96% of bulk phishing attacks, and 76% of targeted attacks. Major breaches at companies like Twilio and Uber involved compromised credentials that MFA would have prevented.

## Deep Dive
### Intro

MFA adds an additional layer of security beyond passwords. TOTP (Time-based One-Time Password) is the most common approach.

### Totp implementation

```php
<?php
class TotpAuthenticator
{
    private const PERIOD = 30;  // Seconds per code
    private const DIGITS = 6;
    
    public function generateSecret(int $length = 20): string
    {
        // Generate random secret
        $secret = random_bytes($length);
        return $this->base32Encode($secret);
    }
    
    public function getQrCodeUri(string $secret, string $email, string $issuer): string
    {
        return sprintf(
            'otpauth://totp/%s:%s?secret=%s&issuer=%s&algorithm=SHA1&digits=%d&period=%d',
            rawurlencode($issuer),
            rawurlencode($email),
            $secret,
            rawurlencode($issuer),
            self::DIGITS,
            self::PERIOD
        );
    }
    
    public function verify(string $secret, string $code, int $window = 1): bool
    {
        $timestamp = time();
        
        // Check current and adjacent time windows
        for ($i = -$window; $i <= $window; $i++) {
            $checkTime = $timestamp + ($i * self::PERIOD);
            $expectedCode = $this->generateCode($secret, $checkTime);
            
            if (hash_equals($expectedCode, $code)) {
                return true;
            }
        }
        
        return false;
    }
    
    private function generateCode(string $secret, int $timestamp): string
    {
        $counter = intdiv($timestamp, self::PERIOD);
        $counterBytes = pack('J', $counter);  // 64-bit big-endian
        
        $hash = hash_hmac('sha1', $counterBytes, $this->base32Decode($secret), true);
        $offset = ord($hash[19]) & 0x0f;
        
        $code = (
            (ord($hash[$offset]) & 0x7f) << 24 |
            (ord($hash[$offset + 1]) & 0xff) << 16 |
            (ord($hash[$offset + 2]) & 0xff) << 8 |
            (ord($hash[$offset + 3]) & 0xff)
        ) % (10 ** self::DIGITS);
        
        return str_pad((string) $code, self::DIGITS, '0', STR_PAD_LEFT);
    }
    
    private function base32Encode(string $data): string { /* ... */ }
    private function base32Decode(string $data): string { /* ... */ }
}
```

### Backup codes

```php
<?php
class BackupCodes
{
    private const CODE_COUNT = 10;
    private const CODE_LENGTH = 8;
    
    public function generate(): array
    {
        $codes = [];
        
        for ($i = 0; $i < self::CODE_COUNT; $i++) {
            $codes[] = $this->generateCode();
        }
        
        return $codes;
    }
    
    private function generateCode(): string
    {
        return strtoupper(bin2hex(random_bytes(self::CODE_LENGTH / 2)));
    }
    
    public function store(int $userId, array $codes): void
    {
        // Hash codes before storage
        foreach ($codes as $code) {
            $hash = password_hash($code, PASSWORD_DEFAULT);
            $this->db->insert('backup_codes', [
                'user_id' => $userId,
                'code_hash' => $hash,
                'used' => false,
            ]);
        }
    }
    
    public function verify(int $userId, string $code): bool
    {
        $codes = $this->db->query(
            'SELECT id, code_hash FROM backup_codes WHERE user_id = ? AND used = 0',
            [$userId]
        );
        
        foreach ($codes as $storedCode) {
            if (password_verify($code, $storedCode['code_hash'])) {
                // Mark as used
                $this->db->update('backup_codes', 
                    ['used' => true],
                    ['id' => $storedCode['id']]
                );
                return true;
            }
        }
        
        return false;
    }
}
```

### Complete mfa flow

```php
<?php
class AuthController
{
    public function login(Request $request): Response
    {
        $user = $this->validateCredentials(
            $request->get('email'),
            $request->get('password')
        );
        
        if (!$user) {
            return $this->error('Invalid credentials');
        }
        
        // Check if MFA is enabled
        if ($user->mfa_enabled) {
            // Create temporary session for MFA verification
            $_SESSION['mfa_user_id'] = $user->id;
            $_SESSION['mfa_expires'] = time() + 300;  // 5 minutes
            
            return $this->redirect('/mfa-verify');
        }
        
        // No MFA - complete login
        $this->completeLogin($user);
        return $this->redirect('/dashboard');
    }
    
    public function verifyMfa(Request $request): Response
    {
        // Check temporary session
        if (!isset($_SESSION['mfa_user_id']) || 
            $_SESSION['mfa_expires'] < time()) {
            return $this->redirect('/login');
        }
        
        $userId = $_SESSION['mfa_user_id'];
        $user = $this->userRepo->find($userId);
        $code = $request->get('code');
        
        // Try TOTP first
        if ($this->totp->verify($user->mfa_secret, $code)) {
            unset($_SESSION['mfa_user_id'], $_SESSION['mfa_expires']);
            $this->completeLogin($user);
            return $this->redirect('/dashboard');
        }
        
        // Try backup code
        if ($this->backupCodes->verify($userId, $code)) {
            unset($_SESSION['mfa_user_id'], $_SESSION['mfa_expires']);
            $this->completeLogin($user);
            return $this->redirect('/dashboard');
        }
        
        return $this->error('Invalid code');
    }
}
```

## Common Pitfalls
1. **Storing MFA secrets in plain text** — The TOTP shared secret must be encrypted at rest. If the database is compromised, plain text secrets allow attackers to generate valid codes.
2. **Not providing backup codes** — Users who lose their MFA device get locked out permanently if no recovery mechanism exists. Always generate and display backup codes during MFA setup.

## Best Practices
1. **Hash backup codes with `password_hash()`** — Treat backup codes like passwords. Hash them individually so a database breach doesn't expose them.
2. **Implement rate limiting on MFA verification** — Limit failed MFA attempts to prevent brute-force attacks on 6-digit codes (only 1 million possibilities per 30-second window).

## Summary
- TOTP is the standard MFA protocol, generating 6-digit codes every 30 seconds based on a shared secret.
- Hash backup codes with `password_hash()` and encrypt TOTP secrets at rest.
- Rate-limit MFA verification attempts to prevent brute-force attacks.

## Resources

- [TOTP RFC](https://datatracker.ietf.org/doc/html/rfc6238) — TOTP algorithm specification

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*