---
source_course: "php-api-development"
source_lesson: "php-api-development-error-handling"
---

# Consistent Error Handling

Consistent error responses make your API predictable and developer-friendly.

## Error Response Format

```php
<?php
class ApiException extends Exception {
    public function __construct(
        string $message,
        public int $statusCode = 400,
        public ?array $errors = null,
        public ?string $code = null
    ) {
        parent::__construct($message);
    }
    
    public function toArray(): array {
        $response = [
            'error' => [
                'message' => $this->getMessage(),
                'code' => $this->code ?? $this->getDefaultCode(),
            ],
        ];
        
        if ($this->errors) {
            $response['error']['details'] = $this->errors;
        }
        
        return $response;
    }
    
    private function getDefaultCode(): string {
        return match($this->statusCode) {
            400 => 'BAD_REQUEST',
            401 => 'UNAUTHORIZED',
            403 => 'FORBIDDEN',
            404 => 'NOT_FOUND',
            422 => 'VALIDATION_ERROR',
            429 => 'RATE_LIMITED',
            500 => 'INTERNAL_ERROR',
            default => 'ERROR',
        };
    }
}

class ValidationException extends ApiException {
    public function __construct(array $errors) {
        parent::__construct('Validation failed', 422, $errors, 'VALIDATION_ERROR');
    }
}

class NotFoundException extends ApiException {
    public function __construct(string $resource = 'Resource') {
        parent::__construct("$resource not found", 404, null, 'NOT_FOUND');
    }
}

class UnauthorizedException extends ApiException {
    public function __construct(string $message = 'Authentication required') {
        parent::__construct($message, 401, null, 'UNAUTHORIZED');
    }
}
```

## Global Error Handler

```php
<?php
class ErrorHandler {
    public static function register(): void {
        set_exception_handler([self::class, 'handleException']);
        set_error_handler([self::class, 'handleError']);
        register_shutdown_function([self::class, 'handleShutdown']);
    }
    
    public static function handleException(Throwable $e): void {
        if ($e instanceof ApiException) {
            self::sendResponse($e->statusCode, $e->toArray());
        } elseif ($e instanceof JsonException) {
            self::sendResponse(400, [
                'error' => [
                    'message' => 'Invalid JSON',
                    'code' => 'INVALID_JSON',
                ],
            ]);
        } else {
            // Log the actual error
            error_log($e->getMessage() . "\n" . $e->getTraceAsString());
            
            // Send generic error (don't expose internals)
            $message = getenv('APP_DEBUG') ? $e->getMessage() : 'An unexpected error occurred';
            self::sendResponse(500, [
                'error' => [
                    'message' => $message,
                    'code' => 'INTERNAL_ERROR',
                ],
            ]);
        }
    }
    
    public static function handleError(int $errno, string $errstr, string $errfile, int $errline): bool {
        throw new ErrorException($errstr, 0, $errno, $errfile, $errline);
    }
    
    public static function handleShutdown(): void {
        $error = error_get_last();
        if ($error && in_array($error['type'], [E_ERROR, E_CORE_ERROR, E_COMPILE_ERROR, E_PARSE])) {
            self::handleException(new ErrorException(
                $error['message'], 0, $error['type'], $error['file'], $error['line']
            ));
        }
    }
    
    private static function sendResponse(int $status, array $body): void {
        http_response_code($status);
        header('Content-Type: application/json');
        echo json_encode($body);
        exit;
    }
}

// Usage
ErrorHandler::register();

// In controller
throw new NotFoundException('User');
// Returns: {"error":{"message":"User not found","code":"NOT_FOUND"}}

throw new ValidationException([
    'email' => 'Invalid email format',
    'password' => 'Password too short',
]);
// Returns: {"error":{"message":"Validation failed","code":"VALIDATION_ERROR","details":{...}}}
```

## Code Examples

**RFC 7807 compliant error handling**

```php
<?php
declare(strict_types=1);

// Complete API error handling example

// Error response structure (RFC 7807 - Problem Details)
class ProblemDetails {
    public function __construct(
        public string $type,
        public string $title,
        public int $status,
        public ?string $detail = null,
        public ?string $instance = null,
        public array $extensions = []
    ) {}
    
    public function toArray(): array {
        $result = [
            'type' => $this->type,
            'title' => $this->title,
            'status' => $this->status,
        ];
        
        if ($this->detail) $result['detail'] = $this->detail;
        if ($this->instance) $result['instance'] = $this->instance;
        
        return array_merge($result, $this->extensions);
    }
    
    public static function validation(array $errors): self {
        return new self(
            type: 'https://api.example.com/problems/validation-error',
            title: 'Validation Failed',
            status: 422,
            detail: 'One or more fields failed validation',
            extensions: ['errors' => $errors]
        );
    }
    
    public static function notFound(string $resource, string|int $id): self {
        return new self(
            type: 'https://api.example.com/problems/not-found',
            title: 'Resource Not Found',
            status: 404,
            detail: "$resource with ID $id was not found",
            instance: "/{$resource}s/$id"
        );
    }
    
    public static function unauthorized(): self {
        return new self(
            type: 'https://api.example.com/problems/unauthorized',
            title: 'Unauthorized',
            status: 401,
            detail: 'Valid authentication credentials are required'
        );
    }
    
    public static function forbidden(): self {
        return new self(
            type: 'https://api.example.com/problems/forbidden',
            title: 'Forbidden',
            status: 403,
            detail: 'You do not have permission to access this resource'
        );
    }
    
    public static function rateLimited(int $retryAfter): self {
        return new self(
            type: 'https://api.example.com/problems/rate-limited',
            title: 'Too Many Requests',
            status: 429,
            detail: 'Rate limit exceeded',
            extensions: ['retryAfter' => $retryAfter]
        );
    }
}

// Usage in controller
class UserController extends Controller {
    public function show(string $id): never {
        $user = $this->users->find((int) $id);
        
        if (!$user) {
            $problem = ProblemDetails::notFound('User', $id);
            $this->problemResponse($problem);
        }
        
        $this->json(['data' => $user->toArray()]);
    }
    
    public function store(): never {
        $data = $this->input();
        $errors = $this->validate($data);
        
        if ($errors) {
            $problem = ProblemDetails::validation($errors);
            $this->problemResponse($problem);
        }
        
        $user = $this->users->create($data);
        $this->json(['data' => $user->toArray()], 201);
    }
    
    protected function problemResponse(ProblemDetails $problem): never {
        http_response_code($problem->status);
        header('Content-Type: application/problem+json');
        echo json_encode($problem->toArray());
        exit;
    }
}
?>
```


## Resources

- [RFC 7807 - Problem Details](https://datatracker.ietf.org/doc/html/rfc7807) — Standard for HTTP API error responses

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*