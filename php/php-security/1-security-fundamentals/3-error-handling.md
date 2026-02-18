---
source_course: "php-security"
source_lesson: "php-security-secure-error-handling"
---

# Secure Error Handling

Proper error handling prevents information leakage while providing useful debugging capabilities.

## Custom Error Handler

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

## Custom Exception Handler

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

## Structured Error Logging

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

## API Error Responses

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

## Resources

- [Error Handling](https://www.php.net/manual/en/book.errorfunc.php) — PHP error handling functions

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*