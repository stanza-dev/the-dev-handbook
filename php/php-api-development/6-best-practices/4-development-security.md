---
source_course: "php-api-development"
source_lesson: "php-api-development-security"
---

# API Security Hardening

## Introduction
Authentication and authorization are only the first layer of API security. Production APIs also need input validation, injection prevention, HTTPS enforcement, and security headers. This lesson covers the essential hardening techniques that protect your API from common attack vectors.

## Key Concepts
- **Input Validation**: Verifying that incoming data matches expected formats, types, and ranges before processing it.
- **SQL Injection**: An attack where malicious SQL is inserted through user input to manipulate database queries.
- **Content-Security-Policy (CSP)**: An HTTP header that controls which resources the browser is allowed to load, preventing XSS attacks.
- **X-Content-Type-Options**: An HTTP header that prevents browsers from MIME-sniffing a response away from the declared content type.
- **HTTPS Enforcement**: Ensuring all API communication happens over encrypted TLS connections.

## Real World Context
In 2023, the OWASP Top 10 for APIs listed injection, broken authentication, and excessive data exposure as the most critical threats. A single unvalidated input field can lead to a full database breach. Security headers add defense-in-depth layers that protect against entire classes of attacks with minimal code.

## Deep Dive

### Input Validation

Never trust client input. Validate every parameter, field, and header before using it in business logic or database queries.

Here is a reusable input validator class:

```php
<?php
class InputValidator
{
    private array $errors = [];

    public function requireString(string $field, mixed $value, int $minLength = 1, int $maxLength = 255): ?string
    {
        if (!is_string($value) || strlen(trim($value)) < $minLength) {
            $this->errors[] = [
                'field' => $field,
                'message' => "{$field} must be at least {$minLength} characters",
            ];
            return null;
        }

        $trimmed = trim($value);
        if (strlen($trimmed) > $maxLength) {
            $this->errors[] = [
                'field' => $field,
                'message' => "{$field} must not exceed {$maxLength} characters",
            ];
            return null;
        }

        return $trimmed;
    }

    public function requireEmail(string $field, mixed $value): ?string
    {
        $email = filter_var($value, FILTER_VALIDATE_EMAIL);
        if ($email === false) {
            $this->errors[] = [
                'field' => $field,
                'message' => "{$field} must be a valid email address",
            ];
            return null;
        }
        return $email;
    }

    public function requireInt(string $field, mixed $value, int $min = PHP_INT_MIN, int $max = PHP_INT_MAX): ?int
    {
        $int = filter_var($value, FILTER_VALIDATE_INT);
        if ($int === false || $int < $min || $int > $max) {
            $this->errors[] = [
                'field' => $field,
                'message' => "{$field} must be an integer between {$min} and {$max}",
            ];
            return null;
        }
        return $int;
    }

    public function hasErrors(): bool
    {
        return !empty($this->errors);
    }

    public function getErrors(): array
    {
        return $this->errors;
    }
}

// Usage in a controller
$body = json_decode(file_get_contents('php://input'), true) ?? [];
$validator = new InputValidator();

$name = $validator->requireString('name', $body['name'] ?? null);
$email = $validator->requireEmail('email', $body['email'] ?? null);
$age = $validator->requireInt('age', $body['age'] ?? null, min: 0, max: 150);

if ($validator->hasErrors()) {
    throw new ValidationException('Invalid input', $validator->getErrors());
}
```

The validator collects all errors at once rather than failing on the first one. This lets the client fix all issues in a single round trip instead of discovering them one at a time.

### SQL Injection Prevention

SQL injection is the most dangerous attack against database-backed APIs. The solution is simple: always use parameterized queries. Never concatenate user input into SQL strings.

Here is the difference between vulnerable and safe code:

```php
<?php
// DANGEROUS: SQL injection vulnerability
$id = $_GET['id'];
$stmt = $pdo->query("SELECT * FROM products WHERE id = $id");
// An attacker sends: ?id=1; DROP TABLE products;--

// SAFE: Parameterized query
$stmt = $pdo->prepare('SELECT * FROM products WHERE id = :id');
$stmt->execute([':id' => $_GET['id']]);
$product = $stmt->fetch(PDO::FETCH_ASSOC);

// SAFE: Using positional parameters
$stmt = $pdo->prepare('SELECT * FROM products WHERE name LIKE ? AND price < ?');
$stmt->execute(["%{$_GET['search']}%", $_GET['max_price']]);
$results = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

Parameterized queries separate SQL structure from data. The database engine treats parameters as values, not as SQL code, making injection impossible.

### Security Headers

Security headers add defense-in-depth against cross-site scripting (XSS), clickjacking, and MIME-type attacks. Here is a middleware that sets all essential security headers:

```php
<?php
class SecurityHeadersMiddleware
{
    public function handle(callable $next): void
    {
        // Prevent MIME-type sniffing
        header('X-Content-Type-Options: nosniff');

        // Prevent clickjacking
        header('X-Frame-Options: DENY');

        // Enable XSS filter (legacy browsers)
        header('X-XSS-Protection: 1; mode=block');

        // Enforce HTTPS for 1 year
        header('Strict-Transport-Security: max-age=31536000; includeSubDomains');

        // Content Security Policy: restrict resource loading
        header("Content-Security-Policy: default-src 'none'; frame-ancestors 'none'");

        // Prevent referrer leakage
        header('Referrer-Policy: strict-origin-when-cross-origin');

        // Control browser features
        header('Permissions-Policy: camera=(), microphone=(), geolocation=()');

        $next();
    }
}
```

For an API that only serves JSON, `Content-Security-Policy: default-src 'none'` is the strictest possible setting because APIs do not load scripts, styles, or images. The `X-Content-Type-Options: nosniff` header prevents browsers from interpreting a JSON response as HTML, which could enable XSS.

### HTTPS Enforcement

Redirect all HTTP requests to HTTPS and set the `Strict-Transport-Security` header so browsers remember to always use HTTPS:

```php
<?php
function enforceHttps(): void
{
    // Check if the request is already HTTPS
    $isHttps = (
        (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off')
        || ($_SERVER['SERVER_PORT'] ?? 0) === 443
        || ($_SERVER['HTTP_X_FORWARDED_PROTO'] ?? '') === 'https'
    );

    if (!$isHttps && ($_ENV['APP_ENV'] ?? '') === 'production') {
        $redirectUrl = 'https://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'];
        header('Location: ' . $redirectUrl, true, 301);
        exit;
    }
}

// Call early in your bootstrap
enforceHttps();
```

The `X-Forwarded-Proto` check handles requests behind a reverse proxy or load balancer where TLS termination happens upstream.

## Common Pitfalls
1. **Validating only on the client side** — Client-side validation improves UX but provides zero security. Attackers bypass JavaScript validation trivially with cURL or Postman. Always validate server-side.
2. **Using string concatenation for SQL** — Even with `addslashes()` or `mysqli_real_escape_string()`, string concatenation is fragile. Parameterized queries are the only reliable defense against SQL injection.

## Best Practices
1. **Validate early, validate everything** — Validate input at the API boundary before it reaches business logic or the database. Reject invalid requests with clear 422 responses.
2. **Apply security headers globally** — Use middleware to set security headers on every response. This ensures no endpoint accidentally misses protection.

## Summary
- Validate all input server-side using type-safe validation before processing or storing data.
- Always use parameterized queries (PDO prepared statements) to prevent SQL injection.
- Set security headers (`X-Content-Type-Options`, `Content-Security-Policy`, `Strict-Transport-Security`) on every API response.
- Enforce HTTPS in production and handle `X-Forwarded-Proto` for proxied environments.
- Collect all validation errors at once so clients can fix everything in a single round trip.

## Code Examples

**Security headers middleware and parameterized queries — two essential layers that protect against XSS, clickjacking, MIME sniffing, and SQL injection**

```php
<?php
// Security headers middleware for API responses
class SecurityHeadersMiddleware
{
    public function handle(callable $next): void
    {
        header('X-Content-Type-Options: nosniff');     // No MIME sniffing
        header('X-Frame-Options: DENY');                // No iframes
        header('Strict-Transport-Security: max-age=31536000; includeSubDomains');
        header("Content-Security-Policy: default-src 'none'; frame-ancestors 'none'");
        header('Referrer-Policy: strict-origin-when-cross-origin');
        header('Permissions-Policy: camera=(), microphone=(), geolocation=()');

        $next();
    }
}

// Parameterized query to prevent SQL injection
$stmt = $pdo->prepare(
    'SELECT * FROM products WHERE category = :category AND price <= :max_price'
);
$stmt->execute([
    ':category' => $userInput['category'],
    ':max_price' => $userInput['max_price'],
]);
$products = $stmt->fetchAll(PDO::FETCH_ASSOC);
```


## Resources

- [PHP PDO Prepared Statements](https://www.php.net/manual/en/pdo.prepared-statements.php) — Official PHP documentation for prepared statements that prevent SQL injection
- [PHP filter_var — Input Filtering](https://www.php.net/manual/en/function.filter-var.php) — Official PHP documentation for input validation and sanitization functions

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*