---
source_course: "php-api-development"
source_lesson: "php-api-development-response-formatting"
---

# Response Formatting

## Introduction
A consistent response format is just as important as consistent URL design. When every endpoint returns data in the same shape, clients can write generic parsing logic instead of handling each endpoint as a special case. This lesson covers header management, JSON envelope patterns, error formatting, and how PHP 8.5's `clone()` with property overrides supports immutable response objects.

## Key Concepts
- **Response envelope**: A wrapper object that places the actual data under a `data` key and errors under an `error` key, creating a predictable top-level structure.
- **Content-Type header**: The HTTP header that tells the client the media type of the response body. For JSON APIs: `application/json; charset=utf-8`.
- **Immutable response object**: A response representation that cannot be modified in place. Instead, you create a new instance with the desired changes. This prevents accidental mutations during middleware processing.

## Real World Context
APIs like the Stripe API and the JSON:API specification use envelope patterns so clients always know where to find data, metadata, and errors. When every endpoint returns `{ "data": ... }` for success and `{ "error": { "message": ... } }` for failure, front-end developers can write a single response handler that works across the entire API.

## Deep Dive

### Setting Headers Correctly
Headers must be set before any output is sent. A well-structured response helper ensures this:

```php
<?php
declare(strict_types=1);

function sendJson(mixed $body, int $status = 200, array $headers = []): never {
    // Set status code first
    http_response_code($status);

    // Default headers
    header('Content-Type: application/json; charset=utf-8');
    header('X-Content-Type-Options: nosniff');

    // Custom headers (e.g., Location, X-Request-Id)
    foreach ($headers as $name => $value) {
        header("{$name}: {$value}");
    }

    // Encode and send
    if ($status !== 204) {
        echo json_encode($body, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
    }

    exit;
}

// 200 with data
sendJson(['data' => ['id' => 1, 'name' => 'Alice']]);

// 201 with Location header
sendJson(['data' => ['id' => 99]], 201, ['Location' => '/users/99']);

// 204 No Content (empty body)
sendJson(null, 204);
```

The `X-Content-Type-Options: nosniff` header prevents browsers from MIME-sniffing the response, which is a basic security measure.

### The Envelope Pattern
Wrap every response in a consistent envelope so clients always know the shape:

```php
<?php
declare(strict_types=1);

class ApiResponse {
    /**
     * Success response with data.
     */
    public static function ok(mixed $data, array $meta = []): array {
        $response = ['data' => $data];
        if ($meta !== []) {
            $response['meta'] = $meta;
        }
        return $response;
    }

    /**
     * Error response with a message and optional field-level details.
     */
    public static function error(string $message, ?array $details = null): array {
        $error = ['message' => $message];
        if ($details !== null) {
            $error['details'] = $details;
        }
        return ['error' => $error];
    }
}

// Success: {"data": {"id": 1, "name": "Alice"}, "meta": {"page": 1}}
$payload = ApiResponse::ok(['id' => 1, 'name' => 'Alice'], ['page' => 1]);

// Error: {"error": {"message": "Validation failed", "details": {"email": "Invalid"}}}
$payload = ApiResponse::error('Validation failed', ['email' => 'Invalid format']);
```

The `data` and `error` keys are mutually exclusive. A response is either a success or an error, never both.

### Error Response Formatting
Structure error responses so clients can display field-level messages:

```php
<?php
declare(strict_types=1);

// A well-structured error response
$errorResponse = [
    'error' => [
        'message' => 'Validation failed',          // Human-readable summary
        'code'    => 'VALIDATION_ERROR',            // Machine-readable error code
        'details' => [                              // Field-level errors
            'email' => 'Must be a valid email address',
            'name'  => 'Must be between 2 and 100 characters',
        ],
    ],
];

http_response_code(422);
header('Content-Type: application/json; charset=utf-8');
echo json_encode($errorResponse, JSON_UNESCAPED_UNICODE);
```

Including a machine-readable `code` alongside the `message` lets clients switch on the code for programmatic error handling while displaying the message to humans.

### PHP 8.5 clone() with Property Overrides for Immutable Responses
PHP 8.5 enhances `clone` with a property override syntax, letting you create modified copies of objects without mutating the original. This is ideal for response objects that pass through middleware:

```php
<?php
declare(strict_types=1);

readonly class Response {
    public function __construct(
        public readonly int $status,
        public readonly array $headers,
        public readonly string $body,
    ) {}
}

// Create an initial response
$response = new Response(
    status: 200,
    headers: ['Content-Type' => 'application/json'],
    body: json_encode(['data' => ['id' => 1]]),
);

// PHP 8.5: create a modified copy without mutating the original
$withCors = clone($response, [
    'headers' => array_merge($response->headers, [
        'Access-Control-Allow-Origin' => '*',
        'Access-Control-Allow-Methods' => 'GET, POST, PUT, DELETE',
    ]),
]);

// $response is unchanged — $withCors has the added CORS headers.
// This is safe for middleware chains where multiple layers modify the response.
```

The `clone()` syntax with property overrides replaces the need for manual `clone` + property reassignment, and it works with `readonly` properties. Each middleware step produces a new response, leaving the original untouched.

## Common Pitfalls
1. **Inconsistent envelope shapes** — Returning `{"user": {...}}` from one endpoint and `{"data": {...}}` from another forces clients to handle each endpoint differently. Standardise on one envelope shape from day one.
2. **Sending output before headers** — If you `echo` anything before calling `header()`, PHP will emit a "headers already sent" warning and the headers will be lost. Always set headers before writing to the output buffer.

## Best Practices
1. **Always set Content-Type** — Even though most clients assume JSON, explicitly setting `Content-Type: application/json; charset=utf-8` prevents parsing ambiguity and satisfies strict HTTP clients.
2. **Include a machine-readable error code** — In addition to a human-readable `message`, add a `code` field like `VALIDATION_ERROR` or `NOT_FOUND`. This lets clients programmatically handle errors without parsing message strings.

## Summary
- Set `Content-Type` and security headers on every response before sending the body.
- Use an envelope pattern: `{"data": ...}` for success, `{"error": {"message": ..., "details": ...}}` for errors.
- Include field-level details in error responses so clients know exactly what to fix.
- PHP 8.5's `clone()` with property overrides enables immutable response objects that are safe for middleware chains.
- Consistency in response formatting is more important than any specific format choice.

## Code Examples

**An immutable JsonResponse helper with envelope pattern, header management, and convenience methods for common status codes**

```php
<?php
declare(strict_types=1);

/**
 * A complete response helper combining envelope pattern,
 * header management, and consistent formatting.
 */
class JsonResponse {
    private int $status = 200;
    private array $headers = [
        'Content-Type' => 'application/json; charset=utf-8',
        'X-Content-Type-Options' => 'nosniff',
    ];

    public function withStatus(int $status): self {
        $clone = clone $this;
        $clone->status = $status;
        return $clone;
    }

    public function withHeader(string $name, string $value): self {
        $clone = clone $this;
        $clone->headers[$name] = $value;
        return $clone;
    }

    public function send(mixed $body): never {
        http_response_code($this->status);

        foreach ($this->headers as $name => $value) {
            header("{$name}: {$value}");
        }

        if ($this->status !== 204 && $body !== null) {
            echo json_encode($body, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
        }

        exit;
    }

    // --- Convenience methods ---

    public function ok(mixed $data, array $meta = []): never {
        $payload = ['data' => $data];
        if ($meta !== []) {
            $payload['meta'] = $meta;
        }
        $this->withStatus(200)->send($payload);
    }

    public function created(mixed $data, string $location): never {
        $this->withStatus(201)
             ->withHeader('Location', $location)
             ->send(['data' => $data]);
    }

    public function noContent(): never {
        $this->withStatus(204)->send(null);
    }

    public function error(string $message, int $status = 400, ?array $details = null): never {
        $error = ['message' => $message];
        if ($details !== null) {
            $error['details'] = $details;
        }
        $this->withStatus($status)->send(['error' => $error]);
    }
}

// Usage in a controller:
$response = new JsonResponse();

// Success
$response->ok(['id' => 1, 'name' => 'Alice'], ['page' => 1]);

// Created
$response->created(['id' => 99, 'name' => 'Bob'], '/users/99');

// Validation error
$response->error('Validation failed', 422, ['email' => 'Invalid format']);
?>
```


## Resources

- [header — PHP Manual](https://www.php.net/manual/en/function.header.php) — Official PHP reference for sending HTTP headers
- [http_response_code — PHP Manual](https://www.php.net/manual/en/function.http-response-code.php) — PHP reference for setting the HTTP response status code

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*