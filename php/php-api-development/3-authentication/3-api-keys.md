---
source_course: "php-api-development"
source_lesson: "php-api-development-api-keys"
---

# API Key Authentication

## Introduction
API keys are the simplest form of API authentication. They work like a password for your application: the client sends a long random string with every request, and the server checks whether that string is valid. While less flexible than JWT, API keys are perfect for server-to-server communication, third-party integrations, and rate-limiting by client.

## Key Concepts
- **API Key**: A long, randomly generated string that identifies a client application.
- **Key Hashing**: The practice of storing only a cryptographic hash of the key, never the raw key itself.
- **Timing-safe Comparison**: Using `hash_equals()` instead of `===` to compare secrets, preventing timing attacks.
- **Key Prefix**: A non-secret prefix (e.g., `sk_live_`) that helps identify key types without revealing the secret portion.

## Real World Context
Services like Stripe, Twilio, and SendGrid all use API keys as their primary authentication method. When you integrate with any third-party service, you will almost certainly use an API key. Understanding how to generate, store, and validate them securely is essential for any PHP API developer.

## Deep Dive

### Generating Secure API Keys

A good API key must be long, random, and unpredictable. PHP’s `random_bytes()` function uses the operating system’s cryptographically secure random number generator.

Here is a complete API key generation service:

```php
<?php
class ApiKeyService {
    public function __construct(
        private readonly \PDO $pdo,
    ) {}

    public function generate(int $clientId, string $label): array {
        // Generate a 32-byte (256-bit) random key
        $rawKey = bin2hex(random_bytes(32)); // 64 hex characters

        // Add a prefix so developers can identify the key type
        $prefixedKey = 'sk_live_' . $rawKey;

        // Store ONLY the hash — never the raw key
        $keyHash = hash('sha256', $prefixedKey);

        $stmt = $this->pdo->prepare(
            'INSERT INTO api_keys (client_id, label, key_hash, created_at)
             VALUES (?, ?, ?, NOW())'
        );
        $stmt->execute([$clientId, $label, $keyHash]);

        // Return the raw key ONCE — it cannot be recovered later
        return [
            'key'   => $prefixedKey,
            'label' => $label,
            'note'  => 'Store this key securely. It will not be shown again.',
        ];
    }
}
```

The raw key is returned exactly once during creation. After that, only the hash exists in the database. If a client loses their key, they must generate a new one.

### Validating API Keys with Timing-Safe Comparison

When validating an incoming key, you must use `hash_equals()` instead of `===`. The `===` operator can leak information through timing differences: it returns faster when the first character differs than when only the last character differs. Attackers can exploit this to guess a key character by character.

Here is how to validate an API key securely:

```php
<?php
class ApiKeyAuthenticator {
    public function __construct(
        private readonly \PDO $pdo,
    ) {}

    public function authenticate(string $rawKey): ?array {
        if ($rawKey === '') {
            return null;
        }

        // Hash the incoming key the same way we hashed it during generation
        $keyHash = hash('sha256', $rawKey);

        $stmt = $this->pdo->prepare(
            'SELECT client_id, label, revoked_at FROM api_keys WHERE key_hash = ?'
        );
        $stmt->execute([$keyHash]);
        $record = $stmt->fetch(\PDO::FETCH_ASSOC);

        if (!$record) {
            return null; // No matching key
        }

        if ($record['revoked_at'] !== null) {
            return null; // Key has been revoked
        }

        return [
            'client_id' => (int) $record['client_id'],
            'label'     => $record['label'],
        ];
    }
}
```

Because we compare hashes (via the SQL query), timing attacks against the raw key are already mitigated. However, if you ever compare raw secrets directly in PHP, always use `hash_equals()`.

Here is the timing-safe comparison function in action:

```php
<?php
// WRONG: vulnerable to timing attacks
if ($incomingSecret === $storedSecret) { /* ... */ }

// CORRECT: timing-safe comparison
if (hash_equals($storedSecret, $incomingSecret)) { /* ... */ }
```

`hash_equals()` always takes the same amount of time regardless of where the strings differ, making timing attacks impossible.

### Reading the API Key from a Request

Clients can send API keys in different locations. The most common approaches are a custom header or the `Authorization` header:

```php
<?php
function extractApiKey(): ?string {
    // Option 1: Custom header (X-API-Key)
    $apiKey = $_SERVER['HTTP_X_API_KEY'] ?? null;
    if ($apiKey !== null) {
        return $apiKey;
    }

    // Option 2: Authorization header with Bearer scheme
    $authHeader = $_SERVER['HTTP_AUTHORIZATION'] ?? '';
    if (preg_match('/^Bearer\s+(sk_live_\S+)$/', $authHeader, $matches)) {
        return $matches[1];
    }

    return null;
}

// Usage in a request handler
$apiKey = extractApiKey();
if ($apiKey === null) {
    http_response_code(401);
    echo json_encode(['error' => 'API key required']);
    exit;
}

$authenticator = new ApiKeyAuthenticator($pdo);
$client = $authenticator->authenticate($apiKey);

if ($client === null) {
    http_response_code(401);
    echo json_encode(['error' => 'Invalid or revoked API key']);
    exit;
}

// Authenticated — $client['client_id'] identifies the caller
```

Supporting both header styles gives your API consumers flexibility while keeping the authentication logic in one place.

## Common Pitfalls
1. **Storing raw API keys in the database** — If your database is breached, every client’s key is immediately compromised. Always store only the SHA-256 hash of the key.
2. **Using `===` to compare secrets** — String comparison with `===` is not timing-safe. Use `hash_equals()` whenever you compare sensitive values directly in PHP code.

## Best Practices
1. **Support key revocation** — Add a `revoked_at` column to your API keys table. When a key is compromised, revoke it immediately without affecting other clients. Never delete key records — keep them for audit trails.
2. **Use descriptive key prefixes** — Prefixes like `sk_live_` and `sk_test_` let developers instantly recognize what environment a key belongs to, reducing the chance of accidentally using production keys in testing.

## Summary
- API keys are long random strings that identify a client application.
- Generate keys with `random_bytes()` and store only the SHA-256 hash in the database.
- Always use `hash_equals()` for timing-safe comparison of secret values.
- Support key revocation and use descriptive prefixes like `sk_live_` for developer experience.
- Return the raw key only once during creation — it cannot be recovered from the hash.

## Code Examples

**Generating, hashing, and validating API keys — raw keys are returned once and only hashes are stored in the database**

```php
<?php
// Generating a secure API key
$rawKey = 'sk_live_' . bin2hex(random_bytes(32));
$keyHash = hash('sha256', $rawKey);

// Store $keyHash in the database, return $rawKey to the client once

// Validating an incoming key
$incomingHash = hash('sha256', $incomingKey);
$stmt = $pdo->prepare('SELECT * FROM api_keys WHERE key_hash = ?');
$stmt->execute([$incomingHash]);
$record = $stmt->fetch();

// Timing-safe comparison for direct secret comparison
if (hash_equals($storedSecret, $userInput)) {
    // Secrets match
}
```


## Resources

- [PHP random_bytes function](https://www.php.net/manual/en/function.random-bytes.php) — Cryptographically secure random byte generator for key generation
- [PHP hash_equals function](https://www.php.net/manual/en/function.hash-equals.php) — Timing-safe string comparison to prevent side-channel attacks

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*