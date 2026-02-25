---
source_course: "php-security"
source_lesson: "php-security-digital-signatures"
---

# Digital Signatures & HMAC

## Introduction
Digital signatures and HMAC (Hash-based Message Authentication Code) verify that data has not been tampered with and authenticate the sender. They are essential for API security, webhook verification, and data integrity.

## Key Concepts
- **HMAC**: A keyed hash that combines a secret key with data to produce a signature. Only someone with the same key can produce the same signature.
- **Digital Signature**: A cryptographic proof that data was created by the holder of a private key, verifiable by anyone with the corresponding public key.
- **Data Integrity**: Assurance that data has not been modified in transit or storage.

## Real World Context
Every Stripe webhook, GitHub webhook, and AWS API request uses HMAC signatures. When Stripe sends a payment notification to your server, it signs the payload with a shared secret. Your server verifies the signature to ensure the request really came from Stripe and wasn't modified by a man-in-the-middle.

## Deep Dive
HMAC uses a shared secret to create a hash that proves authenticity:

```php
<?php
// Creating an HMAC signature
$payload = json_encode(['amount' => 100, 'currency' => 'usd']);
$secret = $_ENV['WEBHOOK_SECRET'];

$signature = hash_hmac('sha256', $payload, $secret);
// Output: a hex string like '5d41402abc4b2a76b9719d...' 
```

The `hash_hmac()` function takes the algorithm, data, and secret key. Only someone with the same secret can generate the same signature.

To verify an incoming webhook, compare the provided signature with one you compute:

```php
<?php
function verifyWebhook(string $payload, string $signature, string $secret): bool {
    $expected = hash_hmac('sha256', $payload, $secret);
    // Use timing-safe comparison to prevent timing attacks
    return hash_equals($expected, $signature);
}

// Verify incoming Stripe-style webhook
$payload = file_get_contents('php://input');
$headerSig = $_SERVER['HTTP_X_SIGNATURE'] ?? '';

if (!verifyWebhook($payload, $headerSig, $_ENV['WEBHOOK_SECRET'])) {
    http_response_code(403);
    die('Invalid signature');
}
```

Always use `hash_equals()` for signature comparison to prevent timing attacks. A regular `===` comparison leaks information about how many characters match.

For asymmetric digital signatures (useful when the verifier shouldn't have the signing key):

```php
<?php
// Sign with private key
$data = 'Important message';
$privateKey = openssl_pkey_get_private(file_get_contents('private.pem'));
openssl_sign($data, $signature, $privateKey, OPENSSL_ALGO_SHA256);

// Verify with public key (anyone can verify)
$publicKey = openssl_pkey_get_public(file_get_contents('public.pem'));
$valid = openssl_verify($data, $signature, $publicKey, OPENSSL_ALGO_SHA256);
// $valid === 1 means signature is valid
```

Asymmetric signatures are used for JWT tokens, code signing, and inter-service authentication where you want to verify without sharing the signing key.

## Common Pitfalls
1. **Using regular string comparison for signatures** — Using `===` instead of `hash_equals()` makes your verification vulnerable to timing attacks. An attacker can determine the correct signature character by character by measuring response times.
2. **Using MD5 or SHA1 for HMAC** — While HMAC-MD5 is theoretically still secure, always use SHA-256 or SHA-512 for new implementations. MD5 and SHA1 have known weaknesses.

## Best Practices
1. **Always use hash_equals() for signature verification** — This function runs in constant time regardless of where strings differ, preventing timing-based attacks.
2. **Store signing keys in environment variables** — Never hardcode secrets in your source code. Use `$_ENV` or a secrets manager.

## Summary
- HMAC creates a keyed hash for data authentication and integrity verification.
- Use `hash_hmac('sha256', $data, $secret)` to generate signatures.
- Always verify signatures with `hash_equals()` to prevent timing attacks.
- Asymmetric signatures (`openssl_sign`/`openssl_verify`) let anyone verify without the signing key.

## Code Examples

**Webhook signature verification using HMAC-SHA256 with timing-safe comparison — the standard pattern for Stripe, GitHub, and other webhook providers**

```php
<?php
// Webhook signature verification (Stripe-style)
function verifyWebhookSignature(
    string $payload,
    string $signatureHeader,
    string $secret
): bool {
    $expected = hash_hmac('sha256', $payload, $secret);
    return hash_equals($expected, $signatureHeader);
}

// Usage in webhook endpoint
$payload = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_WEBHOOK_SIGNATURE'] ?? '';

if (!verifyWebhookSignature($payload, $signature, $_ENV['WEBHOOK_SECRET'])) {
    http_response_code(403);
    exit('Invalid signature');
}

$event = json_decode($payload, true);
// Process verified webhook...
```


## Resources

- [hash_hmac()](https://www.php.net/manual/en/function.hash-hmac.php) — PHP HMAC hash generation function

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*