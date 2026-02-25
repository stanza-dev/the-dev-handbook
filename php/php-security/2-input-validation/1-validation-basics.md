---
source_course: "php-security"
source_lesson: "php-security-validation-basics"
---

# Input Validation Fundamentals

## Introduction
**Never trust user input.** This is the golden rule of web security. Every piece of data from users must be validated.

## Key Concepts
- **filter_var()**: PHP's built-in validation and sanitization function with type-specific filters.
- **Whitelist Validation**: Explicitly defining allowed values and rejecting everything else.
- **Input Boundaries**: Enforcing length, range, and format constraints on all user input.
- **Multi-layer Validation**: Validating on both client-side (UX) and server-side (security).

## Real World Context
Input validation failures are behind most injection attacks. The 2017 Equifax breach exploited an unvalidated input field in Apache Struts. Every form field, URL parameter, HTTP header, and file upload in your PHP application is a potential attack vector.

## Deep Dive
### Intro

**Never trust user input.** This is the golden rule of web security. Every piece of data from users must be validated.

### Validation vs sanitization

- **Validation**: Check if data meets requirements (reject if invalid)
- **Sanitization**: Clean/modify data to make it safe

```php
<?php
// Validation - reject bad data
$email = $_POST['email'];
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    throw new ValidationException('Invalid email format');
}

// Sanitization - clean the data
$email = filter_var($_POST['email'], FILTER_SANITIZE_EMAIL);
```

### Filter functions

### Validation Filters

```php
<?php
// Returns filtered value or false if invalid
filter_var($email, FILTER_VALIDATE_EMAIL);
filter_var($url, FILTER_VALIDATE_URL);
filter_var($ip, FILTER_VALIDATE_IP);
filter_var($int, FILTER_VALIDATE_INT);
filter_var($float, FILTER_VALIDATE_FLOAT);
filter_var($bool, FILTER_VALIDATE_BOOL);

// With options
$age = filter_var($input, FILTER_VALIDATE_INT, [
    'options' => [
        'min_range' => 0,
        'max_range' => 120
    ]
]);

// Directly from superglobals
$page = filter_input(INPUT_GET, 'page', FILTER_VALIDATE_INT);
$search = filter_input(INPUT_POST, 'search', FILTER_SANITIZE_FULL_SPECIAL_CHARS);
```

### Sanitization Filters

```php
<?php
// Remove/encode dangerous characters
filter_var($input, FILTER_SANITIZE_EMAIL);
filter_var($input, FILTER_SANITIZE_URL);
filter_var($input, FILTER_SANITIZE_NUMBER_INT);
filter_var($input, FILTER_SANITIZE_FULL_SPECIAL_CHARS);  // HTML encode
```

### Whitelist validation

Always prefer whitelist over blacklist:

```php
<?php
// BAD: Blacklist (incomplete)
$forbidden = ['<script>', 'javascript:', 'onerror'];
if (str_contains_any($input, $forbidden)) {
    // Attackers can bypass this!
}

// GOOD: Whitelist (explicit allowed values)
$allowedStatuses = ['pending', 'approved', 'rejected'];
$status = $_POST['status'];

if (!in_array($status, $allowedStatuses, true)) {
    throw new ValidationException('Invalid status');
}

// GOOD: Pattern matching for formats
if (!preg_match('/^[a-zA-Z0-9_]{3,20}$/', $username)) {
    throw new ValidationException('Invalid username format');
}
```

### Type coercion dangers

```php
<?php
// PHP's loose comparison is dangerous
$password = $_POST['password'];

// BAD: Type juggling attack
if ($password == $storedPassword) { /* vulnerable */ }

// GOOD: Strict comparison
if ($password === $storedPassword) { /* safe */ }

// Better: Use password_verify()
if (password_verify($password, $hash)) { /* best */ }
```

## Common Pitfalls
1. **Relying on client-side validation alone** — JavaScript validation is for UX only. Attackers bypass it trivially with curl or browser dev tools.
2. **Using blacklist validation** — Trying to block known bad patterns always misses new attack vectors. Whitelist validation is fundamentally more secure.

## Best Practices
1. **Validate at system boundaries** — Validate all input at the point it enters your application (controllers, API handlers) before it reaches business logic.
2. **Use PHP's filter extension** — `filter_var()` with `FILTER_VALIDATE_*` constants provides battle-tested validation for emails, URLs, IPs, and more.

## Summary
- Always validate and sanitize all input using whitelist (allowlist) validation.
- Use `filter_var()` with appropriate `FILTER_VALIDATE_*` filters for type-specific validation.
- Never trust client-side validation alone — always validate server-side.

## Code Examples

**Comprehensive input validator class**

```php
<?php
declare(strict_types=1);

class InputValidator {
    private array $errors = [];
    private array $validated = [];
    
    public function validate(array $rules, array $data): bool {
        foreach ($rules as $field => $fieldRules) {
            $value = $data[$field] ?? null;
            
            foreach ($fieldRules as $rule => $param) {
                if (!$this->applyRule($field, $value, $rule, $param)) {
                    break; // Stop on first error for this field
                }
            }
            
            if (!isset($this->errors[$field])) {
                $this->validated[$field] = $value;
            }
        }
        
        return empty($this->errors);
    }
    
    private function applyRule(string $field, mixed $value, string $rule, mixed $param): bool {
        return match($rule) {
            'required' => $this->validateRequired($field, $value),
            'email' => $this->validateEmail($field, $value),
            'min' => $this->validateMin($field, $value, $param),
            'max' => $this->validateMax($field, $value, $param),
            'in' => $this->validateIn($field, $value, $param),
            'regex' => $this->validateRegex($field, $value, $param),
            default => true,
        };
    }
    
    private function validateRequired(string $field, mixed $value): bool {
        if (empty($value) && $value !== '0') {
            $this->errors[$field] = "$field is required";
            return false;
        }
        return true;
    }
    
    private function validateEmail(string $field, mixed $value): bool {
        if ($value && !filter_var($value, FILTER_VALIDATE_EMAIL)) {
            $this->errors[$field] = "$field must be a valid email";
            return false;
        }
        return true;
    }
    
    private function validateIn(string $field, mixed $value, array $allowed): bool {
        if ($value && !in_array($value, $allowed, true)) {
            $this->errors[$field] = "$field contains an invalid value";
            return false;
        }
        return true;
    }
    
    public function getErrors(): array { return $this->errors; }
    public function getValidated(): array { return $this->validated; }
}

// Usage
$validator = new InputValidator();
$valid = $validator->validate([
    'email' => ['required' => true, 'email' => true],
    'status' => ['required' => true, 'in' => ['active', 'inactive']],
    'age' => ['min' => 18, 'max' => 120],
], $_POST);

if (!$valid) {
    $errors = $validator->getErrors();
}
?>
```


## Resources

- [Filter Functions](https://www.php.net/manual/en/book.filter.php) — PHP filter extension for validation and sanitization

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*