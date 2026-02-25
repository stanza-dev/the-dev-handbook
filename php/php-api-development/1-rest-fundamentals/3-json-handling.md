---
source_course: "php-api-development"
source_lesson: "php-api-development-json-handling"
---

# JSON Request and Response Handling

## Introduction
JSON (JavaScript Object Notation) is the standard data format for REST APIs. PHP provides built-in functions for converting between PHP arrays and JSON strings. In this lesson you will learn how to read JSON request bodies, send JSON responses, and use PHP 8.5's pipe operator to transform data elegantly before encoding.

## Key Concepts
- **json_encode()**: Converts a PHP value (array, object, scalar) into a JSON string.
- **json_decode()**: Parses a JSON string into a PHP array or object.
- **Content-Type header**: Tells the client (or server) what format the body is in. For JSON APIs this is `application/json`.
- **php://input**: A read-only stream that gives you the raw request body, which is where JSON payloads arrive in POST/PUT/PATCH requests.

## Real World Context
Every modern API client library — from JavaScript's `fetch()` to Python's `requests` — sends and expects JSON by default. If your PHP API cannot reliably decode incoming JSON and produce well-formed JSON responses, integration with any front-end or third-party service will fail.

## Deep Dive

### Reading a JSON Request Body
PHP does not automatically parse JSON bodies the way it parses form data. You need to read `php://input` manually:

```php
<?php
declare(strict_types=1);

// Read the raw body once
$rawBody = file_get_contents('php://input');

// Decode with JSON_THROW_ON_ERROR to get an exception on bad input
try {
    $data = json_decode($rawBody, associative: true, flags: JSON_THROW_ON_ERROR);
} catch (\JsonException $e) {
    http_response_code(400);
    echo json_encode(['error' => 'Invalid JSON: ' . $e->getMessage()]);
    exit;
}

$name  = $data['name']  ?? null;
$email = $data['email'] ?? null;
```

Using `JSON_THROW_ON_ERROR` is the modern approach. It converts parse failures into exceptions rather than requiring you to check `json_last_error()` after every call.

### Sending a JSON Response
A well-behaved API sets the `Content-Type` header and uses consistent encoding flags:

```php
<?php
declare(strict_types=1);

function jsonResponse(mixed $data, int $status = 200): never {
    http_response_code($status);
    header('Content-Type: application/json; charset=utf-8');
    echo json_encode(
        $data,
        JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES
    );
    exit;
}

// Return a single user
jsonResponse(['data' => ['id' => 1, 'name' => 'Alice']], 200);

// Return a creation result
jsonResponse(['data' => ['id' => 99, 'name' => 'Bob']], 201);
```

The `JSON_UNESCAPED_UNICODE` flag keeps emoji and international characters readable. `JSON_UNESCAPED_SLASHES` prevents forward slashes in URLs from being escaped.

### Common Encoding Flags
Here are the flags you will use most often:

```php
<?php
// JSON_THROW_ON_ERROR   — throw JsonException instead of returning false
// JSON_UNESCAPED_UNICODE — keep é, ñ, 🎉 as-is instead of \uXXXX
// JSON_UNESCAPED_SLASHES — keep / instead of \/
// JSON_PRETTY_PRINT      — add whitespace (useful for debugging, avoid in production)

$product = ['id' => 5, 'name' => 'Café Blend', 'url' => '/products/5'];

// Production flags
$json = json_encode($product, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
// Output: {"id":5,"name":"Café Blend","url":"/products/5"}
```

These three flags together produce clean, readable JSON.

### PHP 8.5 Pipe Operator for Data Transformation
PHP 8.5 introduces the pipe operator (`|>`), which passes the result of the left expression as the first argument to the function on the right. This is perfect for building response pipelines:

```php
<?php
declare(strict_types=1);

function filterSensitiveFields(array $user): array {
    unset($user['password_hash'], $user['internal_notes']);
    return $user;
}

function addTimestamp(array $payload): array {
    $payload['_meta'] = ['generated_at' => date('c')];
    return $payload;
}

function toJsonString(array $payload): string {
    return json_encode($payload, JSON_THROW_ON_ERROR | JSON_UNESCAPED_SLASHES);
}

// PHP 8.5 pipe operator chains the transformations left-to-right
$user = [
    'id' => 7,
    'name' => 'Alice',
    'password_hash' => '$2y$10$abc...',
    'internal_notes' => 'VIP customer',
];

$responseBody = $user
    |> filterSensitiveFields(...)
    |> addTimestamp(...)
    |> toJsonString(...);

// $responseBody: {"id":7,"name":"Alice","_meta":{"generated_at":"2026-02-25T10:00:00+00:00"}}
echo $responseBody;
```

Each step is a small, testable function. The pipe operator makes the flow obvious without nesting function calls.

## Common Pitfalls
1. **Forgetting the Content-Type header** — If you omit `Content-Type: application/json`, some clients will treat the response as plain text and fail to parse it automatically. Always set the header before echoing JSON.
2. **Using json_decode without error handling** — By default `json_decode` returns `null` on failure, which is the same value you get from decoding `"null"`. Always pass `JSON_THROW_ON_ERROR` so invalid input raises a `JsonException` you can catch.

## Best Practices
1. **Always use JSON_THROW_ON_ERROR** — This flag was added in PHP 7.3 and turns silent failures into catchable exceptions. It is the only reliable way to detect malformed JSON.
2. **Create a single response helper** — Centralise your JSON response logic in one function or class so every endpoint uses the same status-code and header logic. This eliminates inconsistencies across your API.

## Summary
- Use `file_get_contents('php://input')` to read raw JSON request bodies.
- Always decode with `JSON_THROW_ON_ERROR` to catch parse errors as exceptions.
- Set `Content-Type: application/json; charset=utf-8` on every response.
- Combine `JSON_UNESCAPED_UNICODE` and `JSON_UNESCAPED_SLASHES` for clean output.
- PHP 8.5's pipe operator (`|>`) lets you chain data transformations in a readable left-to-right pipeline.

## Code Examples

**A reusable JsonApi helper class that centralises JSON decoding, success responses, and error responses with proper headers and status codes**

```php
<?php
declare(strict_types=1);

// A reusable JSON request/response helper class
class JsonApi {
    /**
     * Decode the incoming JSON body.
     * Throws a JsonException if the body is not valid JSON.
     */
    public static function input(): array {
        $raw = file_get_contents('php://input');

        if ($raw === '' || $raw === false) {
            return [];
        }

        return json_decode($raw, associative: true, flags: JSON_THROW_ON_ERROR);
    }

    /**
     * Send a JSON success response and terminate.
     */
    public static function success(mixed $data, int $status = 200): never {
        self::send(['data' => $data], $status);
    }

    /**
     * Send a JSON error response and terminate.
     */
    public static function error(string $message, int $status = 400): never {
        self::send(['error' => ['message' => $message]], $status);
    }

    private static function send(mixed $payload, int $status): never {
        http_response_code($status);
        header('Content-Type: application/json; charset=utf-8');
        echo json_encode(
            $payload,
            JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES
        );
        exit;
    }
}

// Usage in a request handler:
try {
    $body = JsonApi::input();
    // Process $body...
    JsonApi::success(['id' => 1, 'name' => $body['name'] ?? 'Unknown'], 201);
} catch (\JsonException $e) {
    JsonApi::error('Invalid JSON: ' . $e->getMessage(), 400);
}
?>
```


## Resources

- [json_decode — PHP Manual](https://www.php.net/manual/en/function.json-decode.php) — Official PHP reference for parsing JSON strings into PHP values
- [json_encode — PHP Manual](https://www.php.net/manual/en/function.json-encode.php) — Official PHP reference for encoding PHP values as JSON

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*