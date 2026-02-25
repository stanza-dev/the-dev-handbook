---
source_course: "php-api-development"
source_lesson: "php-api-development-jwt-auth"
---

# JWT Authentication in PHP

## Introduction
JSON Web Tokens (JWT) are the industry standard for stateless API authentication. A JWT carries all the information the server needs to verify a request without touching a database. In this lesson you will learn the three-part structure of a JWT, how to create and verify tokens using the `firebase/php-jwt` library, and how PHP 8.5’s readonly classes with `clone()` property overrides make token payloads safer to work with.

## Key Concepts
- **JWT (JSON Web Token)**: A compact, URL-safe token format consisting of three Base64URL-encoded parts separated by dots.
- **Header**: The first part of a JWT. It declares the signing algorithm (e.g., HS256) and the token type.
- **Payload**: The second part. It contains *claims* — key-value pairs like `user_id`, `exp` (expiration), and `iat` (issued at).
- **Signature**: The third part. A cryptographic hash that proves the header and payload have not been tampered with.
- **Claim**: A piece of information asserted about the subject. Standard claims include `iss` (issuer), `exp` (expiration time), and `sub` (subject).

## Real World Context
Almost every modern API you interact with — from payment processors to social media platforms — uses JWTs for authentication. They are lightweight, self-contained, and horizontally scalable because no server-side session storage is required. If you build APIs in PHP, you will work with JWTs.

## Deep Dive

### JWT Structure

A JWT looks like three Base64URL strings separated by dots:

```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.D4FG5k2nRqH8xM...
|______ Header ______|.|_____ Payload _____|.|___ Signature ___|  
```

The header and payload are just JSON objects encoded in Base64URL. The signature is computed from those two parts plus a secret key.

Here is what the decoded header and payload look like:

```php
<?php
// Decoded header
$header = [
    'alg' => 'HS256',  // HMAC-SHA256 signing algorithm
    'typ' => 'JWT',    // Token type
];

// Decoded payload (claims)
$payload = [
    'sub'     => '42',           // Subject (user ID)
    'email'   => 'dev@app.com',  // Custom claim
    'role'    => 'editor',       // Custom claim
    'iat'     => 1708963200,     // Issued at (Unix timestamp)
    'exp'     => 1709049600,     // Expires at (24 hours later)
];
```

The `sub`, `iat`, and `exp` claims are registered in the JWT specification. You can add any custom claims you need, but keep payloads small because they travel with every request.

### Creating and Verifying JWTs with firebase/php-jwt

The `firebase/php-jwt` library is the most widely used JWT implementation in PHP. Install it with Composer:

```php
<?php
// composer require firebase/php-jwt

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

$secretKey = $_ENV['JWT_SECRET']; // Store securely, never hardcode

// --- Encoding (creating a token) ---
$payload = [
    'sub'   => (string) $user->id,
    'email' => $user->email,
    'role'  => $user->role,
    'iat'   => time(),
    'exp'   => time() + 3600, // 1 hour
];

$token = JWT::encode($payload, $secretKey, 'HS256');
// Returns: "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOi..."

// --- Decoding (verifying a token) ---
try {
    $decoded = JWT::decode($token, new Key($secretKey, 'HS256'));
    // $decoded is a stdClass with all claims
    echo $decoded->sub;   // "42"
    echo $decoded->email;  // "dev@app.com"
} catch (\Firebase\JWT\ExpiredException $e) {
    http_response_code(401);
    echo json_encode(['error' => 'Token expired']);
} catch (\Exception $e) {
    http_response_code(401);
    echo json_encode(['error' => 'Invalid token']);
}
```

The `JWT::decode` method does three things automatically: it verifies the signature, checks the expiration claim, and decodes the payload. If any step fails, it throws an exception.

### PHP 8.5 Readonly Classes with clone() Property Overrides for Token Payloads

PHP 8.5 enhances `clone()` with property overrides for readonly classes, which is perfect for immutable token payload objects. You can create a typed payload and derive modified copies without mutation:

```php
<?php
// PHP 8.5: readonly class with clone() property overrides
readonly class TokenPayload {
    public function __construct(
        public string $sub,
        public string $email,
        public string $role,
        public int $iat,
        public int $exp,
    ) {}

    public function withExtendedExpiry(int $seconds): static {
        // clone() creates a new instance, changing only specified properties
        return clone($this, [
            'exp' => $this->exp + $seconds,
        ]);
    }

    public function withRole(string $newRole): static {
        return clone($this, [
            'role' => $newRole,
        ]);
    }

    public function toArray(): array {
        return [
            'sub'   => $this->sub,
            'email' => $this->email,
            'role'  => $this->role,
            'iat'   => $this->iat,
            'exp'   => $this->exp,
        ];
    }
}

// Usage
$payload = new TokenPayload(
    sub: '42',
    email: 'dev@app.com',
    role: 'editor',
    iat: time(),
    exp: time() + 3600,
);

// Need an admin token? Clone with a different role:
$adminPayload = $payload->withRole('admin');

// Encode it
$token = JWT::encode($adminPayload->toArray(), $secretKey, 'HS256');
```

The `clone()` with property overrides ensures the original payload is never modified, which prevents accidental privilege escalation bugs.

### Complete Login Endpoint

Here is how all the pieces fit together in a login endpoint:

```php
<?php
use Firebase\JWT\JWT;

$router->post('/auth/login', function (Request $request) use ($jwt, $userRepository) {
    $body = $request->json();
    $email = $body['email'] ?? '';
    $password = $body['password'] ?? '';

    $user = $userRepository->findByEmail($email);

    if (!$user || !password_verify($password, $user->passwordHash)) {
        http_response_code(401);
        return ['error' => 'Invalid credentials'];
    }

    $payload = new TokenPayload(
        sub: (string) $user->id,
        email: $user->email,
        role: $user->role,
        iat: time(),
        exp: time() + 86400, // 24 hours
    );

    $token = JWT::encode($payload->toArray(), $_ENV['JWT_SECRET'], 'HS256');

    return [
        'token'      => $token,
        'token_type' => 'Bearer',
        'expires_in' => 86400,
    ];
});
```

The endpoint validates credentials, builds a typed payload, encodes it as a JWT, and returns it alongside metadata so the client knows when to refresh.

## Common Pitfalls
1. **Hardcoding the secret key** — Never put your JWT secret directly in source code. Use environment variables or a secrets manager. A leaked secret lets attackers forge any token they want.
2. **Storing sensitive data in the payload** — The JWT payload is Base64URL-encoded, not encrypted. Anyone can decode it. Never put passwords, credit card numbers, or other secrets in claims.

## Best Practices
1. **Set short expiration times** — Use 15–60 minute tokens with a refresh token flow. Short-lived tokens limit the damage window if a token is stolen.
2. **Always validate the algorithm** — Pass the expected algorithm explicitly to the decode function (e.g., `new Key($secret, 'HS256')`). This prevents "algorithm confusion" attacks where an attacker switches the alg header to `none`.

## Summary
- A JWT has three parts: header, payload, and signature, separated by dots.
- The `firebase/php-jwt` library handles encoding and decoding with automatic expiration and signature checks.
- PHP 8.5’s readonly classes with `clone()` property overrides provide a safe, immutable way to model token payloads.
- Never hardcode secrets, never store sensitive data in payloads, and always set short expiration times.

## Code Examples

**Creating and verifying a JWT with firebase/php-jwt — the library checks signature, expiration, and structure automatically**

```php
<?php
use Firebase\JWT\JWT;
use Firebase\JWT\Key;

// Encoding a JWT
$secretKey = $_ENV['JWT_SECRET'];
$payload = [
    'sub'   => (string) $user->id,
    'email' => $user->email,
    'role'  => $user->role,
    'iat'   => time(),
    'exp'   => time() + 3600,
];
$token = JWT::encode($payload, $secretKey, 'HS256');

// Decoding and verifying a JWT
try {
    $decoded = JWT::decode($token, new Key($secretKey, 'HS256'));
    // $decoded->sub === "42"
} catch (\Firebase\JWT\ExpiredException $e) {
    // Token has expired — client must re-authenticate
} catch (\Exception $e) {
    // Invalid signature or malformed token
}
```


## Resources

- [PHP hash_hmac function](https://www.php.net/manual/en/function.hash-hmac.php) — PHP documentation for HMAC hashing used in JWT signatures
- [PHP base64_encode function](https://www.php.net/manual/en/function.base64-encode.php) — Base64 encoding used in JWT header and payload encoding

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*