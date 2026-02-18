---
source_course: "php-essentials"
source_lesson: "php-essentials-input-validation"
---

# Input Validation & Sanitization

**Never trust user input!** Always validate and sanitize data before using it.

## Validation vs Sanitization

- **Validation**: Check if data meets requirements (reject if invalid)
- **Sanitization**: Clean/modify data to make it safe

## PHP Filter Functions

```php
<?php
// Validate - returns false if invalid
$email = filter_var($input, FILTER_VALIDATE_EMAIL);
$url = filter_var($input, FILTER_VALIDATE_URL);
$int = filter_var($input, FILTER_VALIDATE_INT);
$float = filter_var($input, FILTER_VALIDATE_FLOAT);
$bool = filter_var($input, FILTER_VALIDATE_BOOL);

// With options
$age = filter_var($input, FILTER_VALIDATE_INT, [
    'options' => ['min_range' => 1, 'max_range' => 120]
]);
```

## Sanitization Filters

```php
<?php
// Sanitize - cleans the data
$email = filter_var($input, FILTER_SANITIZE_EMAIL);
$string = filter_var($input, FILTER_SANITIZE_FULL_SPECIAL_CHARS);
$int = filter_var($input, FILTER_SANITIZE_NUMBER_INT);
$url = filter_var($input, FILTER_SANITIZE_URL);
```

## Filter Input Directly

```php
<?php
// Filter directly from superglobals
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
$page = filter_input(INPUT_GET, 'page', FILTER_VALIDATE_INT);
```

## Preventing XSS (Cross-Site Scripting)

```php
<?php
// ALWAYS escape output
$userInput = '<script>alert("XSS")</script>';

// This is DANGEROUS:
echo $userInput;  // Executes JavaScript!

// This is SAFE:
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');
// Output: &lt;script&gt;alert("XSS")&lt;/script&gt;
```

## Common Validation Patterns

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

## Whitelist Validation

```php
<?php
// Validate against allowed values
$allowedColors = ['red', 'green', 'blue'];
$color = $_POST['color'] ?? '';

if (!in_array($color, $allowedColors, true)) {
    $error = 'Invalid color selected';
}
```

## Code Examples

**Fluent validation class for form handling**

```php
<?php
// Comprehensive input validation class
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

// Usage
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
- [Data Filtering](https://www.php.net/manual/en/filter.filters.php) — Available filter types and options

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*