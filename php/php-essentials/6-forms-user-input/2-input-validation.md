---
source_course: "php-essentials"
source_lesson: "php-essentials-input-validation"
---

# Input Validation & Sanitization

## Introduction

The golden rule of web development is: never trust user input. Every piece of data coming from forms, URLs, or cookies must be validated before use and sanitized before output. This lesson teaches you PHP's built-in filter functions and common validation patterns that protect your application from malicious input.

## Key Concepts

- **Validation**: Checking whether data meets requirements and rejecting it if invalid. The data is not modified.
- **Sanitization**: Cleaning or modifying data to remove potentially dangerous content.
- **`filter_var()`**: PHP's Swiss-army knife for both validation and sanitization using predefined filters.
- **XSS (Cross-Site Scripting)**: An attack where malicious scripts are injected into web pages viewed by other users.
- **`htmlspecialchars()`**: Converts special HTML characters to entities, preventing script injection.

## Real World Context

Every major security breach involving web applications traces back to insufficient input validation. SQL injection, XSS, and command injection all exploit unvalidated input. PHP's `filter_var()` function and `htmlspecialchars()` are your first line of defense. Frameworks like Laravel and Symfony build their validation layers on top of these same primitives.

## Deep Dive

### Validation vs Sanitization

Validation checks data and returns `false` on failure. Sanitization cleans data and always returns a modified string:

- **Validation**: "Is this a valid email?" → Returns the email or `false`.
- **Sanitization**: "Remove dangerous characters from this string" → Returns cleaned string.

### PHP Filter Functions

The `filter_var()` function handles both validation and sanitization:

```php
<?php
// Validate - returns the value if valid, false if invalid
$email = filter_var($input, FILTER_VALIDATE_EMAIL);
$url = filter_var($input, FILTER_VALIDATE_URL);
$int = filter_var($input, FILTER_VALIDATE_INT);
$float = filter_var($input, FILTER_VALIDATE_FLOAT);
$bool = filter_var($input, FILTER_VALIDATE_BOOL);

// With options for range checking
$age = filter_var($input, FILTER_VALIDATE_INT, [
    'options' => ['min_range' => 1, 'max_range' => 120]
]);
```

When validation fails, `filter_var()` returns `false`. Always use `=== false` to check, since valid values like `0` or `""` are also falsy.

### Sanitization Filters

Sanitization filters clean data rather than rejecting it:

```php
<?php
// Sanitize - cleans the data
$email = filter_var($input, FILTER_SANITIZE_EMAIL);
$string = filter_var($input, FILTER_SANITIZE_FULL_SPECIAL_CHARS);
$int = filter_var($input, FILTER_SANITIZE_NUMBER_INT);
$url = filter_var($input, FILTER_SANITIZE_URL);
```

Sanitization is useful as a preprocessing step before validation.

### Filter Input Directly

You can filter directly from superglobals without accessing them manually:

```php
<?php
// Filter directly from superglobals
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
$page = filter_input(INPUT_GET, 'page', FILTER_VALIDATE_INT);
```

This is more concise and handles missing keys gracefully by returning `null`.

### Preventing XSS

Cross-Site Scripting occurs when user input is rendered as HTML without escaping:

```php
<?php
$userInput = '<script>alert("XSS")</script>';

// DANGEROUS - executes JavaScript:
echo $userInput;

// SAFE - converts to HTML entities:
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');
// Output: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;
```

Always use `htmlspecialchars()` with `ENT_QUOTES` and `'UTF-8'` when outputting any user-provided data into HTML.

### Common Validation Patterns

Here is a reusable validation function covering the most common form fields:

```php
<?php
function validateForm(array $data): array {
    $errors = [];
    
    // Required field
    if (empty($data['name'])) {
        $errors['name'] = 'Name is required';
    } elseif (strlen($data['name']) < 2) {
        $errors['name'] = 'Name must be at least 2 characters';
    }
    
    // Email validation
    if (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
        $errors['email'] = 'Invalid email address';
    }
    
    // Password strength
    if (strlen($data['password']) < 8) {
        $errors['password'] = 'Password must be at least 8 characters';
    }
    
    // Numeric range
    $age = filter_var($data['age'], FILTER_VALIDATE_INT);
    if ($age === false || $age < 18 || $age > 120) {
        $errors['age'] = 'Age must be between 18 and 120';
    }
    
    return $errors;
}
```

This pattern returns an associative array of field-specific error messages, making it easy to display errors next to each form field.

### Whitelist Validation

For fields with a fixed set of allowed values, validate against a whitelist:

```php
<?php
$allowedColors = ['red', 'green', 'blue'];
$color = $_POST['color'] ?? '';

if (!in_array($color, $allowedColors, true)) {
    $error = 'Invalid color selected';
}
```

Always use strict comparison (`true` as the third argument) to prevent type juggling.

## Common Pitfalls

1. **Validating but not sanitizing output** — Validation checks input; sanitization protects output. You need both. Validate on the way in, escape on the way out.
2. **Using `strip_tags()` for XSS prevention** — `strip_tags()` is unreliable and can be bypassed. Always use `htmlspecialchars()` instead.
3. **Checking `filter_var()` results with `==` instead of `===`** — Since `filter_var()` returns `false` on failure, and valid values like `0` are also falsy, you must use `=== false` for the comparison.

## Best Practices

1. **Validate input, escape output** — This is the fundamental security principle. Validate data when you receive it; escape it when you render it.
2. **Use `filter_input()` over direct superglobal access** — It handles missing keys gracefully and makes the filtering intent explicit.
3. **Build a reusable validation layer** — Whether a simple function or a class, centralizing validation logic prevents inconsistencies across your application.

## Summary

- Validation rejects bad data; sanitization cleans data. Always do both.
- `filter_var()` with `FILTER_VALIDATE_*` constants validates emails, URLs, integers, and more.
- `htmlspecialchars($input, ENT_QUOTES, 'UTF-8')` is the correct way to prevent XSS.
- `filter_input()` reads and filters superglobals in one step.
- Always validate against whitelists for fields with a fixed set of allowed values.

## Code Examples

**Fluent validation class using method chaining to validate multiple fields in a readable pipeline**

```php
<?php
// Fluent validation class for form handling
class Validator {
    private array $errors = [];
    
    public function required(string $field, mixed $value): self {
        if (empty(trim($value))) {
            $this->errors[$field] = ucfirst($field) . ' is required';
        }
        return $this;
    }
    
    public function email(string $field, string $value): self {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            $this->errors[$field] = 'Invalid email format';
        }
        return $this;
    }
    
    public function minLength(string $field, string $value, int $min): self {
        if (strlen($value) < $min) {
            $this->errors[$field] = "$field must be at least $min characters";
        }
        return $this;
    }
    
    public function isValid(): bool {
        return empty($this->errors);
    }
    
    public function getErrors(): array {
        return $this->errors;
    }
}

// Usage with method chaining
$validator = new Validator();
$validator
    ->required('name', $_POST['name'] ?? '')
    ->email('email', $_POST['email'] ?? '')
    ->minLength('password', $_POST['password'] ?? '', 8);

if ($validator->isValid()) {
    // Process form
} else {
    $errors = $validator->getErrors();
}
?>
```


## Resources

- [Filter Functions](https://www.php.net/manual/en/book.filter.php) — PHP filter extension documentation
- [Data Filtering](https://www.php.net/manual/en/filter.examples.validation.php) — Available filter types and options

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*