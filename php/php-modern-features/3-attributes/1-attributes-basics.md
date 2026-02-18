---
source_course: "php-modern-features"
source_lesson: "php-modern-features-attributes-basics"
---

# Introduction to Attributes (PHP 8.0+)

Attributes are structured metadata that can be attached to classes, methods, properties, parameters, and more. They replace docblock annotations with a native, type-safe solution.

## Basic Syntax

```php
<?php
#[Attribute]
class Route {
    public function __construct(
        public string $path,
        public string $method = 'GET'
    ) {}
}

#[Route('/users', 'GET')]
class UserController {
    #[Route('/users/{id}', 'GET')]
    public function show(int $id): User {}
    
    #[Route('/users', 'POST')]
    public function create(): User {}
}
```

## Built-in Attributes

### #[Override] (PHP 8.3+)

Ensures a method overrides a parent method:

```php
<?php
class Animal {
    public function speak(): string {
        return 'Some sound';
    }
}

class Dog extends Animal {
    #[Override]
    public function speak(): string {  // OK - overrides parent
        return 'Woof!';
    }
    
    #[Override]
    public function bark(): string {   // Error! No parent method
        return 'Bark!';
    }
}
```

### #[Deprecated] (PHP 8.4+)

Mark code as deprecated:

```php
<?php
class Api {
    #[Deprecated('Use newMethod() instead', since: '2.0')]
    public function oldMethod(): void {
        // Legacy code
    }
    
    public function newMethod(): void {
        // New implementation
    }
}
```

### #[SensitiveParameter] (PHP 8.2+)

Hide sensitive data in stack traces:

```php
<?php
function authenticate(
    string $username,
    #[SensitiveParameter] string $password
): bool {
    // If exception occurs, $password won't appear in stack trace
    return verify($username, $password);
}
```

## Attribute Targets

```php
<?php
#[Attribute(Attribute::TARGET_CLASS)]              // Only on classes
#[Attribute(Attribute::TARGET_METHOD)]             // Only on methods
#[Attribute(Attribute::TARGET_PROPERTY)]           // Only on properties
#[Attribute(Attribute::TARGET_PARAMETER)]          // Only on parameters
#[Attribute(Attribute::TARGET_CLASS_CONSTANT)]     // Only on constants
#[Attribute(Attribute::TARGET_ALL)]                // Anywhere (default)

// Combine targets
#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
class MyAttribute {}
```

## Repeatable Attributes

```php
<?php
#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class Middleware {
    public function __construct(public string $name) {}
}

class Controller {
    #[Middleware('auth')]
    #[Middleware('logging')]
    #[Middleware('cache')]
    public function index(): Response {}
}
```

## Code Examples

**Custom validation attributes**

```php
<?php
declare(strict_types=1);

// Validation attributes example
#[Attribute(Attribute::TARGET_PROPERTY)]
class Required {
    public function __construct(public string $message = 'Field is required') {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Email {
    public function __construct(public string $message = 'Invalid email format') {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class MinLength {
    public function __construct(
        public int $min,
        public string $message = 'Too short'
    ) {}
}

// Usage
class UserDTO {
    #[Required]
    #[MinLength(2)]
    public string $name;
    
    #[Required]
    #[Email]
    public string $email;
    
    #[Required]
    #[MinLength(8, message: 'Password must be at least 8 characters')]
    public string $password;
}
?>
```


## Resources

- [Attributes Overview](https://www.php.net/manual/en/language.attributes.overview.php) — Official introduction to PHP attributes

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*