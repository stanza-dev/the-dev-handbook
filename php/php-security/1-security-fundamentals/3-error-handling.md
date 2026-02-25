---
source_course: "php-security"
source_lesson: "php-security-secure-error-handling"
---

# Secure Error Handling

## Introduction
Proper error handling prevents information leakage while providing useful debugging capabilities.

## Key Concepts
- **Error Logging vs Display**: Log errors to files, never display them to users in production.
- **Error Reference IDs**: Return opaque error identifiers to users while logging full details server-side.
- **Custom Error Handlers**: Use `set_error_handler()` and `set_exception_handler()` for centralized error management.
- **Sensitive Data Scrubbing**: Remove passwords, tokens, and PII from log entries before writing.

## Real World Context
Stack traces in error messages have been used in real-world attacks to discover database schemas, file paths, and API keys. Companies like GitHub and Stripe use opaque error IDs (e.g., "ref: abc123") so support teams can look up details without exposing internals to users.

## Deep Dive
### Intro

Proper error handling prevents information leakage while providing useful debugging capabilities.

### Custom error handler

```php
<?php
set_error_handler(function(int $errno, string $errstr, string $errfile, int $errline): bool {
    // Log full details
    error_log(sprintf(
        "[%s] %s in %s:%d",
        match($errno) {
            E_WARNING => 'WARNING',
            E_NOTICE => 'NOTICE',
            E_USER_ERROR => 'USER_ERROR',
            default => 'ERROR',
        },
        $errstr,
        $errfile,
        $errline
    ));
    
    // In production, don't display to user
    if ($_ENV['APP_ENV'] === 'production') {
        return true;  // Don't execute PHP's internal handler
    }
    
    return false;  // Let PHP handle display in dev
});
```

### Custom exception handler

```php
<?php
set_exception_handler(function(Throwable $e): void {
    // Generate unique error ID for tracking
    $errorId = bin2hex(random_bytes(8));
    
    // Log complete error details
    error_log(sprintf(
        "[%s] Uncaught %s: %s in %s:%d\nStack trace:\n%s",
        $errorId,
        get_class($e),
        $e->getMessage(),
        $e->getFile(),
        $e->getLine(),
        $e->getTraceAsString()
    ));
    
    // Show safe error page to user
    http_response_code(500);
    
    if ($_ENV['APP_ENV'] === 'production') {
        echo "An error occurred. Reference: $errorId";
    } else {
        // Show details in development
        echo "<pre>" . htmlspecialchars($e) . "</pre>";
    }
});
```

### Structured error logging

```php
<?php
class SecureLogger
{
    public function logError(Throwable $e, array $context = []): string
    {
        $errorId = $this->generateErrorId();
        
        $logData = [
            'error_id' => $errorId,
            'timestamp' => date('c'),
            'type' => get_class($e),
            'message' => $e->getMessage(),
            'code' => $e->getCode(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'trace' => $e->getTraceAsString(),
            'context' => $this->sanitizeContext($context),
        ];
        
        // Log to secure location
        file_put_contents(
            '/var/log/app/errors.log',
            json_encode($logData) . "\n",
            FILE_APPEND | LOCK_EX
        );
        
        return $errorId;
    }
    
    private function sanitizeContext(array $context): array
    {
        $sensitive = ['password', 'token', 'secret', 'key', 'auth'];
        
        return array_map(function($value, $key) use ($sensitive) {
            foreach ($sensitive as $word) {
                if (stripos($key, $word) !== false) {
                    return '[REDACTED]';
                }
            }
            return $value;
        }, $context, array_keys($context));
    }
}
```

### Api error responses

```php
<?php
class ApiErrorHandler
{
    public function handle(Throwable $e): array
    {
        $errorId = bin2hex(random_bytes(8));
        
        // Log full details internally
        $this->logger->logError($e, ['error_id' => $errorId]);
        
        // Return safe response
        if ($e instanceof ValidationException) {
            return [
                'error' => 'validation_error',
                'message' => $e->getMessage(),
                'errors' => $e->getErrors(),
            ];
        }
        
        if ($e instanceof NotFoundException) {
            return [
                'error' => 'not_found',
                'message' => 'Resource not found',
            ];
        }
        
        // Generic error for everything else
        return [
            'error' => 'server_error',
            'message' => 'An unexpected error occurred',
            'reference' => $errorId,
        ];
    }
}
```

## Common Pitfalls
1. **Logging sensitive data** — Stack traces and context arrays may contain passwords, tokens, or credit card numbers. Always sanitize before logging.
2. **Catching and silencing exceptions** — Empty catch blocks hide bugs and security issues. Always log the exception, even if you handle it gracefully.

## Best Practices
1. **Use structured logging** — Log in JSON format with consistent fields (timestamp, severity, error ID, context) for easier analysis and alerting.
2. **Implement error reference IDs** — Generate a unique ID per error, return it to the user, and log the full details server-side with that ID.

## Summary
- Never display error details to users in production — use error reference IDs instead.
- Implement custom error and exception handlers for consistent, secure error processing.
- Sanitize all log entries to prevent sensitive data leakage.

## Resources

- [Error Handling](https://www.php.net/manual/en/book.errorfunc.php) — PHP error handling functions

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*