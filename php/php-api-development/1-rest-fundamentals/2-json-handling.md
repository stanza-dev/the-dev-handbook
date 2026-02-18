---
source_course: "php-api-development"
source_lesson: "php-api-development-json-handling"
---

# JSON Request & Response Handling

Modern APIs communicate via JSON. PHP provides excellent tools for JSON handling.

## Reading JSON Requests

```php
<?php
// Get raw POST body
$json = file_get_contents('php://input');

// Decode JSON
$data = json_decode($json, associative: true);

if (json_last_error() !== JSON_ERROR_NONE) {
    http_response_code(400);
    echo json_encode(['error' => 'Invalid JSON']);
    exit;
}

// Access data
$name = $data['name'] ?? null;
$email = $data['email'] ?? null;
```

## Sending JSON Responses

```php
<?php
function jsonResponse(mixed $data, int $status = 200): never {
    http_response_code($status);
    header('Content-Type: application/json; charset=utf-8');
    
    echo json_encode($data, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE);
    exit;
}

// Usage
jsonResponse(['id' => 1, 'name' => 'John'], 201);
```

## JSON Encoding Options

```php
<?php
$data = [
    'id' => 1,
    'name' => 'Test',
    'emoji' => '🎉',
    'url' => 'https://example.com/path?q=1',
];

// Default
echo json_encode($data);
// {"id":1,"name":"Test","emoji":"\ud83c\udf89","url":"https:\/\/example.com\/path?q=1"}

// With options
echo json_encode($data, 
    JSON_UNESCAPED_UNICODE |  // Keep emojis readable
    JSON_UNESCAPED_SLASHES |  // Don't escape /
    JSON_PRETTY_PRINT         // Human readable
);
/*
{
    "id": 1,
    "name": "Test",
    "emoji": "🎉",
    "url": "https://example.com/path?q=1"
}
*/
```

## Error Handling

```php
<?php
function decodeJson(string $json): array {
    try {
        return json_decode(
            $json,
            associative: true,
            flags: JSON_THROW_ON_ERROR
        );
    } catch (JsonException $e) {
        throw new BadRequestException('Invalid JSON: ' . $e->getMessage());
    }
}

function encodeJson(mixed $data): string {
    try {
        return json_encode($data, JSON_THROW_ON_ERROR);
    } catch (JsonException $e) {
        throw new RuntimeException('JSON encoding failed: ' . $e->getMessage());
    }
}
```

## Consistent Response Format

```php
<?php
class ApiResponse {
    public static function success(mixed $data, int $status = 200): never {
        self::send(['data' => $data], $status);
    }
    
    public static function created(mixed $data): never {
        self::send(['data' => $data], 201);
    }
    
    public static function noContent(): never {
        http_response_code(204);
        exit;
    }
    
    public static function error(string $message, int $status = 400, ?array $errors = null): never {
        $response = ['error' => ['message' => $message]];
        if ($errors !== null) {
            $response['error']['details'] = $errors;
        }
        self::send($response, $status);
    }
    
    private static function send(array $data, int $status): never {
        http_response_code($status);
        header('Content-Type: application/json; charset=utf-8');
        echo json_encode($data, JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR);
        exit;
    }
}

// Usage
ApiResponse::success(['id' => 1, 'name' => 'John']);
// {"data":{"id":1,"name":"John"}}

ApiResponse::error('Validation failed', 422, ['email' => 'Invalid format']);
// {"error":{"message":"Validation failed","details":{"email":"Invalid format"}}}
```

## Code Examples

**Complete JSON API request/response handler**

```php
<?php
declare(strict_types=1);

// Complete JSON API handler
class JsonApiHandler {
    private array $requestData = [];
    
    public function __construct() {
        $this->parseRequest();
    }
    
    private function parseRequest(): void {
        $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
        
        if (str_contains($contentType, 'application/json')) {
            $json = file_get_contents('php://input');
            
            if ($json !== '' && $json !== '[]' && $json !== '{}') {
                try {
                    $this->requestData = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
                } catch (JsonException $e) {
                    $this->errorResponse('Invalid JSON: ' . $e->getMessage(), 400);
                }
            }
        }
    }
    
    public function input(string $key, mixed $default = null): mixed {
        return $this->requestData[$key] ?? $_GET[$key] ?? $default;
    }
    
    public function all(): array {
        return array_merge($_GET, $this->requestData);
    }
    
    public function validate(array $rules): array {
        $data = [];
        $errors = [];
        
        foreach ($rules as $field => $fieldRules) {
            $value = $this->input($field);
            
            foreach ($fieldRules as $rule) {
                if ($rule === 'required' && ($value === null || $value === '')) {
                    $errors[$field] = "$field is required";
                    break;
                }
                
                if ($rule === 'email' && $value && !filter_var($value, FILTER_VALIDATE_EMAIL)) {
                    $errors[$field] = "$field must be a valid email";
                    break;
                }
                
                if (str_starts_with($rule, 'min:')) {
                    $min = (int) substr($rule, 4);
                    if ($value && strlen($value) < $min) {
                        $errors[$field] = "$field must be at least $min characters";
                        break;
                    }
                }
            }
            
            if (!isset($errors[$field])) {
                $data[$field] = $value;
            }
        }
        
        if ($errors) {
            $this->errorResponse('Validation failed', 422, $errors);
        }
        
        return $data;
    }
    
    public function successResponse(mixed $data, int $status = 200): never {
        $this->sendJson(['data' => $data], $status);
    }
    
    public function errorResponse(string $message, int $status = 400, ?array $details = null): never {
        $error = ['message' => $message];
        if ($details) {
            $error['details'] = $details;
        }
        $this->sendJson(['error' => $error], $status);
    }
    
    private function sendJson(array $data, int $status): never {
        http_response_code($status);
        header('Content-Type: application/json; charset=utf-8');
        echo json_encode($data, JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR);
        exit;
    }
}

// Usage
$api = new JsonApiHandler();

$data = $api->validate([
    'name' => ['required', 'min:2'],
    'email' => ['required', 'email'],
]);

// If validation passes, $data contains validated input
$api->successResponse(['user' => $data], 201);
?>
```


## Resources

- [json_encode](https://www.php.net/manual/en/function.json-encode.php) — PHP JSON encoding documentation

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*