---
source_course: "php-api-development"
source_lesson: "php-api-development-error-handling"
---

# Consistent Error Handling

## Introduction
Inconsistent error responses are one of the biggest frustrations for API consumers. When every endpoint returns errors in a different format, clients cannot build reliable error handling. RFC 7807 "Problem Details for HTTP APIs" provides a standardized error format that every endpoint can follow.

## Key Concepts
- **RFC 7807 Problem Details**: A standard format for HTTP API error responses that includes `type`, `title`, `status`, `detail`, and `instance` fields.
- **Error Envelope**: A consistent wrapper structure for all error responses so clients always know where to find error information.
- **Exception-to-Response Mapping**: A centralized mechanism that catches exceptions and converts them into structured API error responses.
- **Error Transformation Chain**: A pipeline that processes exceptions through multiple handlers to produce the final error response.

## Real World Context
Imagine consuming an API where a 404 returns `{"error": "not found"}`, a 422 returns `{"message": "invalid", "errors": [...]}`, and a 500 returns plain text. Every error format is different. RFC 7807 solves this by defining a single, predictable error shape that clients can handle uniformly.

## Deep Dive

### RFC 7807 Problem Details Format

The Problem Details specification defines a JSON object with five standard fields. Here is a PHP class that implements the format:

```php
<?php
class ProblemDetails implements JsonSerializable
{
    public function __construct(
        public readonly string $type,
        public readonly string $title,
        public readonly int $status,
        public readonly string $detail,
        public readonly ?string $instance = null,
        public readonly array $extensions = [],
    ) {}

    public function jsonSerialize(): array
    {
        $data = [
            'type' => $this->type,
            'title' => $this->title,
            'status' => $this->status,
            'detail' => $this->detail,
        ];

        if ($this->instance !== null) {
            $data['instance'] = $this->instance;
        }

        return array_merge($data, $this->extensions);
    }

    public function send(): never
    {
        http_response_code($this->status);
        header('Content-Type: application/problem+json');
        echo json_encode($this, JSON_UNESCAPED_SLASHES);
        exit;
    }
}
```

The `type` field is a URI that identifies the error category. The `title` is a human-readable summary. The `detail` provides specifics about this particular occurrence. Extensions allow adding custom fields like validation errors.

### Custom Exception Classes

Define exception classes that map directly to HTTP error responses:

```php
<?php
abstract class ApiException extends RuntimeException
{
    abstract public function getStatusCode(): int;
    abstract public function getErrorType(): string;
    abstract public function getTitle(): string;
}

class NotFoundException extends ApiException
{
    public function getStatusCode(): int { return 404; }
    public function getErrorType(): string { return '/errors/not-found'; }
    public function getTitle(): string { return 'Resource Not Found'; }
}

class ValidationException extends ApiException
{
    public function __construct(
        string $message,
        private readonly array $errors = [],
    ) {
        parent::__construct($message);
    }

    public function getStatusCode(): int { return 422; }
    public function getErrorType(): string { return '/errors/validation'; }
    public function getTitle(): string { return 'Validation Error'; }
    public function getErrors(): array { return $this->errors; }
}

class RateLimitException extends ApiException
{
    public function getStatusCode(): int { return 429; }
    public function getErrorType(): string { return '/errors/rate-limit'; }
    public function getTitle(): string { return 'Too Many Requests'; }
}
```

Each exception carries its own HTTP status code and error metadata. This keeps error definitions close to the domain logic that throws them.

### Exception-to-Response Mapping

A global error handler catches all unhandled exceptions and converts them to Problem Details responses:

```php
<?php
function handleApiException(Throwable $exception): never
{
    if ($exception instanceof ApiException) {
        $extensions = [];
        if ($exception instanceof ValidationException) {
            $extensions['errors'] = $exception->getErrors();
        }

        $problem = new ProblemDetails(
            type: $exception->getErrorType(),
            title: $exception->getTitle(),
            status: $exception->getStatusCode(),
            detail: $exception->getMessage(),
            extensions: $extensions,
        );
    } else {
        // Unknown exceptions become 500 errors
        $problem = new ProblemDetails(
            type: '/errors/internal',
            title: 'Internal Server Error',
            status: 500,
            detail: 'An unexpected error occurred',
        );

        // Log the actual exception for debugging
        error_log($exception->getMessage() . "\n" . $exception->getTraceAsString());
    }

    $problem->send();
}

// Register as global exception handler
set_exception_handler('handleApiException');
```

The handler distinguishes between known `ApiException` types and unexpected errors. Known exceptions expose their details to the client, while unknown exceptions return a generic 500 to avoid leaking internal information.

### Error Transformation with the Pipe Operator

PHP 8.5 introduces the pipe operator (`|>`) which enables clean error transformation chains. Instead of nested function calls, you can pipe an exception through a series of transformers:

```php
<?php
// PHP 8.5: Pipe operator for error transformation chains
function enrichWithRequestId(ProblemDetails $problem): ProblemDetails
{
    return new ProblemDetails(
        type: $problem->type,
        title: $problem->title,
        status: $problem->status,
        detail: $problem->detail,
        instance: $problem->instance,
        extensions: array_merge(
            $problem->extensions,
            ['request_id' => bin2hex(random_bytes(8))]
        ),
    );
}

function addTimestamp(ProblemDetails $problem): ProblemDetails
{
    return new ProblemDetails(
        type: $problem->type,
        title: $problem->title,
        status: $problem->status,
        detail: $problem->detail,
        instance: $problem->instance,
        extensions: array_merge(
            $problem->extensions,
            ['timestamp' => gmdate('c')]
        ),
    );
}

// PHP 8.5 pipe operator chains transformations cleanly
$response = $baseProblem
    |> enrichWithRequestId(...)
    |> addTimestamp(...);
```

The pipe operator passes the result of each function as the first argument to the next function. This produces a readable transformation pipeline without deeply nested calls.

## Common Pitfalls
1. **Leaking stack traces in production** — Never expose exception traces or internal file paths in API responses. Log them server-side and return a generic message to the client.
2. **Inconsistent error shapes across endpoints** — If some endpoints return `{"error": "..."}` and others return `{"message": "..."}`, clients cannot build unified error handling. Use a single format (like RFC 7807) everywhere.

## Best Practices
1. **Use `application/problem+json` content type** — RFC 7807 defines this content type specifically for error responses. Setting it tells clients to expect the Problem Details format and enables specialized parsing.
2. **Include a request ID in every error** — A unique `request_id` in error responses lets support teams correlate client-reported errors with server logs for faster debugging.

## Summary
- RFC 7807 Problem Details provides a standardized format for HTTP API error responses with `type`, `title`, `status`, `detail`, and optional extensions.
- Custom exception classes carry their own HTTP status codes and error metadata.
- A global exception handler converts all exceptions into Problem Details responses, keeping error formatting centralized.
- PHP 8.5's pipe operator (`|>`) enables clean error transformation chains without nested function calls.
- Never leak stack traces or internal details in production error responses.

## Code Examples

**RFC 7807 Problem Details error response with validation errors — provides a consistent, standardized error format for all API endpoints**

```php
<?php
// RFC 7807 Problem Details response
$problem = new ProblemDetails(
    type: '/errors/validation',
    title: 'Validation Error',
    status: 422,
    detail: 'The request body contains invalid fields',
    extensions: [
        'errors' => [
            ['field' => 'email', 'message' => 'Invalid email format'],
            ['field' => 'price', 'message' => 'Must be a positive number'],
        ],
    ],
);

// Sends:
// HTTP/1.1 422 Unprocessable Entity
// Content-Type: application/problem+json
// {
//   "type": "/errors/validation",
//   "title": "Validation Error",
//   "status": 422,
//   "detail": "The request body contains invalid fields",
//   "errors": [
//     {"field": "email", "message": "Invalid email format"},
//     {"field": "price", "message": "Must be a positive number"}
//   ]
// }
$problem->send();
```


## Resources

- [PHP set_exception_handler](https://www.php.net/manual/en/function.set-exception-handler.php) — Official PHP documentation for the global exception handler used in centralized error handling

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*