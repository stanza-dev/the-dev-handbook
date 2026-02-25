---
source_course: "php-api-development"
source_lesson: "php-api-development-request-validation"
---

# Request Validation and Sanitization

## Introduction
Never trust client input. Every value arriving in a request — query parameters, JSON body fields, headers — must be validated and sanitised before your application uses it. This lesson covers PHP's built-in validation tools, manual JSON body validation, and how PHP 8.5 features improve the validation story.

## Key Concepts
- **Validation**: Checking that input conforms to expected rules (e.g., an email field contains a valid email address).
- **Sanitization**: Cleaning input by removing or encoding dangerous characters (e.g., stripping HTML tags from a username).
- **filter_input()**: A PHP function that reads a value from a superglobal and applies a validation or sanitization filter in one step.
- **Type coercion**: Converting a string value from the request into the correct PHP type (e.g., `(int) $params['id']`).

## Real World Context
Input validation is the first line of defence against SQL injection, cross-site scripting, and business logic errors. In 2023, OWASP ranked "Injection" and "Broken Access Control" as the top two web application security risks. Both are mitigated by rigorous input validation. Skipping validation does not just cause bugs — it creates security vulnerabilities.

## Deep Dive

### filter_input() for Superglobals
PHP's `filter_input()` function reads a value from `$_GET`, `$_POST`, `$_COOKIE`, or `$_SERVER` and applies a filter in one atomic step:

```php
<?php
declare(strict_types=1);

// Validate and sanitise GET parameters
$page = filter_input(INPUT_GET, 'page', FILTER_VALIDATE_INT, [
    'options' => ['default' => 1, 'min_range' => 1, 'max_range' => 1000],
]);

$email = filter_input(INPUT_GET, 'email', FILTER_VALIDATE_EMAIL);

$search = filter_input(INPUT_GET, 'q', FILTER_SANITIZE_SPECIAL_CHARS);

// $page is guaranteed to be an integer between 1 and 1000, or 1 if missing/invalid
// $email is false if invalid, null if not provided
// $search has special characters converted to HTML entities
```

The advantage over reading `$_GET` directly is that validation and reading happen in one call, reducing the chance of using raw input by accident.

### Validating JSON Request Bodies
For JSON payloads there is no `filter_input()` shortcut. You must decode the body and validate each field manually:

```php
<?php
declare(strict_types=1);

function validateUserInput(array $data): array {
    $errors = [];

    // Required string: name
    if (!isset($data['name']) || !is_string($data['name']) || trim($data['name']) === '') {
        $errors['name'] = 'Name is required and must be a non-empty string';
    } elseif (mb_strlen($data['name']) > 100) {
        $errors['name'] = 'Name must not exceed 100 characters';
    }

    // Required email
    if (!isset($data['email']) || !filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
        $errors['email'] = 'A valid email address is required';
    }

    // Optional integer: age
    if (isset($data['age'])) {
        $age = filter_var($data['age'], FILTER_VALIDATE_INT, [
            'options' => ['min_range' => 0, 'max_range' => 150],
        ]);
        if ($age === false) {
            $errors['age'] = 'Age must be an integer between 0 and 150';
        }
    }

    return $errors;
}

// Usage in a controller
$body   = json_decode(file_get_contents('php://input'), true, flags: JSON_THROW_ON_ERROR);
$errors = validateUserInput($body);

if ($errors !== []) {
    http_response_code(422);
    echo json_encode(['error' => ['message' => 'Validation failed', 'details' => $errors]]);
    exit;
}
```

Return a 422 Unprocessable Entity with a structured error response so the client knows exactly which fields failed and why.

### Type Coercion
Request values arrive as strings. Cast them to the correct types early:

```php
<?php
declare(strict_types=1);

// Route parameters are always strings
$userId = (int) $params['id'];        // "42" → 42
$price  = (float) $body['price'];     // "19.99" → 19.99
$active = filter_var($body['active'], FILTER_VALIDATE_BOOLEAN); // "true" → true
```

Using `filter_var` with `FILTER_VALIDATE_BOOLEAN` correctly handles string values like `"true"`, `"yes"`, `"1"`, `"on"` and their falsy counterparts.

### PHP 8.5 Features for Validation
PHP 8.5 improves validation ergonomics with the pipe operator, allowing you to chain validation steps cleanly:

```php
<?php
declare(strict_types=1);

// PHP 8.5: pipe operator for a validation pipeline
function trimString(string $value): string {
    return trim($value);
}

function requireNonEmpty(string $value): string {
    if ($value === '') {
        throw new \InvalidArgumentException('Value must not be empty');
    }
    return $value;
}

function maxLength(int $max): \Closure {
    return function (string $value) use ($max): string {
        if (mb_strlen($value) > $max) {
            throw new \InvalidArgumentException("Value must not exceed {$max} characters");
        }
        return $value;
    };
}

try {
    $name = $body['name']
        |> trimString(...)
        |> requireNonEmpty(...)
        |> maxLength(100);
} catch (\InvalidArgumentException $e) {
    // Handle validation failure
}
```

Each function in the pipeline transforms or validates the value, and the pipe operator threads the result through in a readable, sequential manner.

## Common Pitfalls
1. **Validating only on the client side** — Client-side validation improves UX but provides zero security. Anyone can send a request directly with `curl`. Always validate on the server.
2. **Using type casts without range checks** — `(int) "abc"` silently produces `0` in PHP. Always validate that the value is numeric before casting, or use `FILTER_VALIDATE_INT` which returns `false` for non-numeric strings.

## Best Practices
1. **Return all validation errors at once** — Do not stop at the first error. Collect every validation failure and return them together so the client can fix all issues in a single retry.
2. **Use 422 for validation failures** — Return 422 Unprocessable Entity (not 400 Bad Request) when the JSON is syntactically valid but the data fails business rules. Reserve 400 for genuinely malformed requests like invalid JSON.

## Summary
- Use `filter_input()` for superglobal values (`$_GET`, `$_POST`) and `filter_var()` for JSON body fields.
- Validate every field: check type, format, range, and required-ness.
- Return 422 with a structured error object listing all failures.
- Cast values to their correct PHP types immediately after validation.
- PHP 8.5's pipe operator enables clean, readable validation pipelines.

## Code Examples

**A fluent RequestValidator class that collects all validation errors and returns a 422 response if any field is invalid**

```php
<?php
declare(strict_types=1);

/**
 * A simple request validator for JSON API bodies.
 * Collects all errors before returning.
 */
class RequestValidator {
    private array $errors = [];
    private array $validated = [];

    public function __construct(private readonly array $data) {}

    public function requireString(string $field, int $maxLength = 255): self {
        $value = $this->data[$field] ?? null;

        if (!is_string($value) || trim($value) === '') {
            $this->errors[$field] = "{$field} is required";
        } elseif (mb_strlen($value) > $maxLength) {
            $this->errors[$field] = "{$field} must not exceed {$maxLength} characters";
        } else {
            $this->validated[$field] = trim($value);
        }

        return $this;
    }

    public function requireEmail(string $field): self {
        $value = $this->data[$field] ?? null;

        if (!is_string($value) || !filter_var($value, FILTER_VALIDATE_EMAIL)) {
            $this->errors[$field] = "{$field} must be a valid email";
        } else {
            $this->validated[$field] = $value;
        }

        return $this;
    }

    public function optionalInt(string $field, int $min = 0, int $max = PHP_INT_MAX): self {
        if (!isset($this->data[$field])) {
            return $this;
        }

        $value = filter_var($this->data[$field], FILTER_VALIDATE_INT, [
            'options' => ['min_range' => $min, 'max_range' => $max],
        ]);

        if ($value === false) {
            $this->errors[$field] = "{$field} must be an integer between {$min} and {$max}";
        } else {
            $this->validated[$field] = $value;
        }

        return $this;
    }

    public function validate(): array {
        if ($this->errors !== []) {
            http_response_code(422);
            header('Content-Type: application/json');
            echo json_encode([
                'error' => [
                    'message' => 'Validation failed',
                    'details' => $this->errors,
                ],
            ]);
            exit;
        }

        return $this->validated;
    }
}

// Usage:
$body = json_decode(file_get_contents('php://input'), true, flags: JSON_THROW_ON_ERROR);

$data = (new RequestValidator($body))
    ->requireString('name', 100)
    ->requireEmail('email')
    ->optionalInt('age', 0, 150)
    ->validate();

// $data is guaranteed to have valid 'name', 'email', and optionally 'age'
http_response_code(201);
echo json_encode(['data' => $data]);
?>
```


## Resources

- [filter_input — PHP Manual](https://www.php.net/manual/en/function.filter-input.php) — Official PHP reference for reading and filtering superglobal values
- [filter_var — PHP Manual](https://www.php.net/manual/en/function.filter-var.php) — PHP reference for validating and sanitising individual values

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*