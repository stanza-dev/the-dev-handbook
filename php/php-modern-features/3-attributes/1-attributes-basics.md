---
source_course: "php-modern-features"
source_lesson: "php-modern-features-attributes-basics"
---

# Introduction to Attributes

## Introduction
Attributes, introduced in PHP 8.0, are structured metadata annotations that can be attached to classes, methods, properties, parameters, and constants. They replace the informal docblock annotation pattern with a native, type-safe, reflectable solution.

## Key Concepts
- **Attribute**: A PHP class decorated with `#[Attribute]` that can be attached to code elements using `#[...]` syntax.
- **Target**: The kind of code element an attribute can be applied to (class, method, property, parameter, constant).
- **Repeatable Attribute**: An attribute that can appear multiple times on the same target.

## Real World Context
Before attributes, PHP frameworks used docblock annotations (`@Route`, `@ORM\Column`) parsed by regex. These were fragile, not validated by PHP, and invisible to the engine. Native attributes are type-checked, IDE-friendly, and reflectable without third-party parsers. Symfony, Laravel, and Doctrine all migrated to native attributes.

## Deep Dive

### Basic Syntax

Define an attribute class and use it:

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

The `#[Attribute]` annotation on the class tells PHP it can be used as an attribute. The `#[Route(...)]` syntax applies it.

### Built-in Attributes

PHP provides several built-in attributes:

#### #[Override] (PHP 8.3+)

Ensures a method actually overrides a parent method:

```php
<?php
class Animal {
    public function speak(): string { return 'Some sound'; }
}

class Dog extends Animal {
    #[Override]
    public function speak(): string { return 'Woof!'; }  // OK
    
    #[Override]
    public function bark(): string { return 'Bark!'; }   // Error! No parent method
}
```

This catches refactoring bugs where the parent method is renamed but child classes are not updated.

#### #[Deprecated] (PHP 8.4+)

Mark code as deprecated with a message:

```php
<?php
class Api {
    #[Deprecated('Use newMethod() instead', since: '2.0')]
    public function oldMethod(): void {}
    
    public function newMethod(): void {}
}
```

PHP 8.5 extends `#[Deprecated]` to traits and constants, not just methods and classes.

#### #[SensitiveParameter] (PHP 8.2+)

Hide sensitive values in stack traces:

```php
<?php
function authenticate(
    string $username,
    #[SensitiveParameter] string $password
): bool {
    return verify($username, $password);
}
```

If an exception occurs, `$password` is replaced with `SensitiveParameterValue` in the stack trace.

#### #[NoDiscard] (PHP 8.5)

Warn when a function's return value is ignored:

```php
<?php
#[NoDiscard]
function computeChecksum(string $data): string {
    return hash('sha256', $data);
}

computeChecksum($data);  // Warning: return value not used
$checksum = computeChecksum($data);  // OK
(void) computeChecksum($data);  // OK — explicitly discarded with (void) cast
```

This is useful for pure functions where ignoring the return value is almost certainly a bug.

### Attribute Targets

Restrict where an attribute can be used:

```php
<?php
#[Attribute(Attribute::TARGET_CLASS)]              // Only on classes
#[Attribute(Attribute::TARGET_METHOD)]             // Only on methods
#[Attribute(Attribute::TARGET_PROPERTY)]           // Only on properties
#[Attribute(Attribute::TARGET_PARAMETER)]          // Only on parameters
#[Attribute(Attribute::TARGET_CLASS_CONSTANT)]     // Only on constants
#[Attribute(Attribute::TARGET_ALL)]                // Anywhere (default)

// Combine targets with bitwise OR
#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
class MyAttribute {}
```

Target restrictions are enforced at the time the attribute is instantiated via reflection.

### Repeatable Attributes

Allow multiple instances on the same target:

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

Without `IS_REPEATABLE`, applying the same attribute twice would cause an error.

## Common Pitfalls
1. **Forgetting `#[Attribute]` on the class** — Without the `#[Attribute]` annotation, PHP will not recognize the class as a valid attribute and will throw an error when you try to use it.
2. **Using attributes without reading them** — Attributes are metadata only. They have no effect unless code reads them via reflection. Simply adding `#[Route('/')]` does nothing without a router that scans for it.

## Best Practices
1. **Set specific targets** — Use `Attribute::TARGET_METHOD` instead of the default `TARGET_ALL` to catch misuse at runtime.
2. **Use constructor promotion in attributes** — Attributes are classes, so `public function __construct(public string $path)` keeps them concise.

## Summary
- Attributes are native metadata annotations using `#[...]` syntax.
- Define attribute classes with `#[Attribute]` and constructor parameters.
- Built-in attributes include `#[Override]`, `#[Deprecated]`, `#[SensitiveParameter]`, and `#[NoDiscard]` (PHP 8.5).
- Control where attributes can appear with target flags.
- Use `IS_REPEATABLE` to allow multiple instances on one target.

## Code Examples

**Custom validation attributes applied to a DTO — a framework would read these via reflection to validate input**

```php
<?php
declare(strict_types=1);

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