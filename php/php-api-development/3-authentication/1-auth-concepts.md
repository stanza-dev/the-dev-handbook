---
source_course: "php-api-development"
source_lesson: "php-api-development-auth-concepts"
---

# Authentication vs Authorization

## Introduction
Every API that serves more than public data must answer two questions: "Who are you?" and "What are you allowed to do?" These two concerns — authentication and authorization — are often confused but solve fundamentally different problems. Understanding the distinction is the first step toward building secure APIs in PHP.

## Key Concepts
- **Authentication (authn)**: The process of verifying *who* the caller is. It answers "Are you really User #42?"
- **Authorization (authz)**: The process of verifying *what* the caller is allowed to do. It answers "Can User #42 delete this post?"
- **API Key**: A long random string the client sends with every request to identify itself.
- **Bearer Token**: A credential (often a JWT) sent in the `Authorization` header.
- **OAuth 2.0**: A delegation framework that lets users grant third-party apps limited access to their resources without sharing passwords.

## Real World Context
Mixing up authentication and authorization leads to security bugs that reach production. For example, checking that a request has a valid token (authn) but forgetting to check that the token’s owner actually has admin privileges (authz) is one of the OWASP Top 10 vulnerabilities — Broken Access Control. Every PHP developer who builds APIs will encounter these concepts daily.

## Deep Dive
Let’s walk through the three most common API authentication strategies and see how each one identifies the caller.

### Strategy 1: API Keys

API keys are the simplest approach. The server generates a random string and gives it to the client. The client includes it in every request.

Here is a basic example of sending an API key via a custom header:

```php
<?php
// Client sends: X-API-Key: sk_live_abc123def456
$apiKey = $_SERVER['HTTP_X_API_KEY'] ?? '';

if ($apiKey === '') {
    http_response_code(401);
    echo json_encode(['error' => 'Missing API key']);
    exit;
}

// Look up the key in the database
$hashedKey = hash('sha256', $apiKey);
$stmt = $pdo->prepare('SELECT * FROM api_keys WHERE key_hash = ?');
$stmt->execute([$hashedKey]);
$keyRecord = $stmt->fetch();

if (!$keyRecord) {
    http_response_code(401);
    echo json_encode(['error' => 'Invalid API key']);
    exit;
}

// $keyRecord now tells us WHO is calling
echo json_encode(['client' => $keyRecord['client_name']]);
```

Notice we never store the raw key — only a SHA-256 hash. This means a database leak does not expose usable credentials.

### Strategy 2: Bearer Tokens (JWT)

Bearer tokens are sent in the `Authorization` header. JWTs are the most popular format because they are self-contained: the server can verify them without a database lookup.

Here is how a client sends a Bearer token and how the server reads it:

```php
<?php
// Client sends: Authorization: Bearer eyJhbGciOiJIUzI1Ni...
$authHeader = $_SERVER['HTTP_AUTHORIZATION'] ?? '';

if (!preg_match('/^Bearer\s+(\S+)$/', $authHeader, $matches)) {
    http_response_code(401);
    echo json_encode(['error' => 'Missing or malformed Bearer token']);
    exit;
}

$token = $matches[1];
// The token is decoded and verified in the next lesson
```

The key advantage is statelessness: the token itself carries the user’s identity, so no session storage is needed on the server.

### Strategy 3: OAuth 2.0

OAuth is more complex. Instead of the user giving their password to a third-party app, they authorize the app through the resource server. The app receives a time-limited access token.

The typical OAuth flow has four steps:

```php
<?php
// Step 1 — Redirect user to the authorization server
$authUrl = 'https://accounts.example.com/authorize?' . http_build_query([
    'client_id'     => $clientId,
    'redirect_uri'  => 'https://myapp.com/callback',
    'response_type' => 'code',
    'scope'         => 'read:profile',
]);
header("Location: $authUrl");
exit;

// Step 2 — The authorization server redirects back with a code
// Step 3 — Exchange the code for an access token (server-to-server)
// Step 4 — Use the access token to call the API
```

OAuth is the standard for "Login with GitHub/Google" flows and is used when your API acts as either a provider or a consumer of third-party identity.

### Authentication vs Authorization in Practice

Once the caller is authenticated, authorization checks happen at the resource level:

```php
<?php
// Authentication happened earlier — we know who the user is
$currentUser = $request->getAuthUser();

// Authorization — can this user delete the post?
$post = $postRepository->find($postId);

if ($post->authorId !== $currentUser->id && $currentUser->role !== 'admin') {
    http_response_code(403); // Forbidden, not 401
    echo json_encode(['error' => 'You cannot delete this post']);
    exit;
}

$postRepository->delete($postId);
echo json_encode(['message' => 'Post deleted']);
```

Note the distinction: a `401 Unauthorized` response means "I don’t know who you are" (authn failure), while a `403 Forbidden` response means "I know who you are, but you’re not allowed" (authz failure).

## Common Pitfalls
1. **Using 403 when you mean 401** — Return 401 when the user is not authenticated at all. Return 403 when authenticated but lacking permission. Mixing these up confuses API consumers and makes debugging harder.
2. **Storing raw API keys in the database** — Always hash API keys before storage. A database breach that exposes raw keys gives attackers immediate access to every client account.

## Best Practices
1. **Use HTTPS everywhere** — Tokens and API keys travel in HTTP headers. Without TLS, they are visible to anyone on the network. Never rely on authentication over plain HTTP.
2. **Separate authn from authz logic** — Keep authentication middleware ("who are you?") separate from authorization checks ("can you do this?"). This makes both easier to test and reuse across routes.

## Summary
- Authentication verifies identity; authorization verifies permissions.
- API keys, Bearer tokens (JWT), and OAuth 2.0 are the three most common API authentication strategies.
- Use 401 for authentication failures and 403 for authorization failures.
- Always hash credentials before storing them and always use HTTPS.
- Keep authentication and authorization logic in separate layers for maintainability.

## Code Examples

**Authentication checks identity (401 on failure), then authorization checks permissions (403 on failure)**

```php
<?php
// Authentication: verify WHO the caller is
$apiKey = $_SERVER['HTTP_X_API_KEY'] ?? '';
$hashedKey = hash('sha256', $apiKey);
// Assuming $apiKeyRepository is injected via constructor
$client = $apiKeyRepository->findByHash($hashedKey);

if (!$client) {
    http_response_code(401); // "I don't know who you are"
    exit;
}

// Authorization: verify WHAT the caller can do
if (!in_array('write:posts', $client->scopes)) {
    http_response_code(403); // "I know you, but you can't do this"
    exit;
}

// Both checks passed — proceed with the request
$postService->create($requestBody);
```


## Resources

- [PHP password_hash function](https://www.php.net/manual/en/function.password-hash.php) — Official PHP documentation for secure password hashing
- [PHP hash_equals function](https://www.php.net/manual/en/function.hash-equals.php) — Timing-safe string comparison for credential verification

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*