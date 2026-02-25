---
source_course: "php-modern-features"
source_lesson: "php-modern-features-constructor-promotion"
---

# Constructor Property Promotion

## Introduction
Constructor property promotion, introduced in PHP 8.0, is a shorthand that declares and initializes class properties directly in the constructor signature. It eliminates one of PHP's most common sources of boilerplate code.

## Key Concepts
- **Promoted Property**: A constructor parameter prefixed with a visibility keyword (`public`, `protected`, `private`) that automatically becomes a class property.
- **Mixed Parameters**: You can combine promoted and regular (non-promoted) parameters in the same constructor.

## Real World Context
Before PHP 8.0, every class property required three steps: declare the property, add a constructor parameter, assign the parameter to the property. For a class with five properties, that meant fifteen lines of repetitive code. Promotion reduces this to five lines, making PHP competitive with languages like Kotlin and TypeScript for concise data classes.

## Deep Dive

### The Old Way

Before promotion, declaring properties required significant repetition:

```php
<?php
class User {
    private string $name;
    private string $email;
    private int $age;
    
    public function __construct(
        string $name,
        string $email,
        int $age
    ) {
        $this->name = $name;
        $this->email = $email;
        $this->age = $age;
    }
}
```

Every property appears three times: declaration, parameter, and assignment.

### The New Way

With promotion, a visibility keyword before the parameter does all three at once:

```php
<?php
class User {
    public function __construct(
        private string $name,
        private string $email,
        private int $age
    ) {}
}
```

The properties are declared, typed, and assigned automatically. The empty constructor body `{}` is all that is needed.

### Mixed Promotion and Regular Parameters

You can combine promoted properties with regular parameters:

```php
<?php
class Post {
    public function __construct(
        private string $title,      // Promoted
        private string $content,    // Promoted
        private Author $author,     // Promoted
        string $tempData = ''       // NOT promoted (no visibility)
    ) {
        $this->processTemp($tempData);
    }
}
```

Only parameters with a visibility keyword become properties. Regular parameters are available in the constructor body but not stored as properties.

### With Default Values

Promoted properties support default values:

```php
<?php
class Config {
    public function __construct(
        public string $environment = 'production',
        public bool $debug = false,
        public int $cacheTime = 3600,
        public array $features = []
    ) {}
}

$config = new Config(debug: true);
echo $config->environment;  // 'production'
echo $config->debug;        // true
```

This pairs perfectly with named arguments to create expressive, self-documenting constructors.

### Combining with Readonly (PHP 8.1+)

Promotion and readonly work together:

```php
<?php
class ImmutableUser {
    public function __construct(
        public readonly string $id,
        public readonly string $name,
        public readonly string $email
    ) {}
}

$user = new ImmutableUser('1', 'John', 'john@example.com');
echo $user->name;  // John
$user->name = 'Jane';  // Error! Cannot modify readonly property
```

This pattern creates immutable value objects with minimal code.

## Common Pitfalls
1. **Assuming all parameters are promoted** — Only parameters with a visibility keyword become properties. Forgetting the keyword means the value is not stored.
2. **Using promotion in non-constructor methods** — Promotion only works in `__construct()`, not in regular methods.

## Best Practices
1. **Use promotion for DTOs and value objects** — Data-heavy classes benefit the most from reduced boilerplate. Combine with `readonly` for immutable data carriers.
2. **Combine with named arguments** — `new User(name: 'Alice', email: 'alice@test.com')` is clearer than positional arguments, especially when the class has optional parameters.

## Summary
- Constructor promotion declares, types, and assigns properties in one step.
- Add a visibility keyword to a constructor parameter to promote it.
- Non-promoted parameters remain regular constructor arguments.
- Combine with `readonly` for concise immutable classes.
- Works perfectly with named arguments for clear, flexible instantiation.

## Code Examples

**Data Transfer Object combining constructor promotion, readonly, defaults, and a named-argument factory method**

```php
<?php
declare(strict_types=1);

class CreateUserDTO {
    public function __construct(
        public readonly string $name,
        public readonly string $email,
        public readonly string $password,
        public readonly ?string $phone = null,
        public readonly array $roles = ['user'],
        public readonly \DateTimeImmutable $createdAt = new \DateTimeImmutable()
    ) {}
    
    public static function fromRequest(array $data): self {
        return new self(
            name: $data['name'] ?? throw new InvalidArgumentException('Name required'),
            email: $data['email'] ?? throw new InvalidArgumentException('Email required'),
            password: $data['password'] ?? throw new InvalidArgumentException('Password required'),
            phone: $data['phone'] ?? null,
            roles: $data['roles'] ?? ['user']
        );
    }
}

$dto = CreateUserDTO::fromRequest($_POST);
echo $dto->name;
?>
```


## Resources

- [Constructor Promotion](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion) — Official documentation for constructor property promotion

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*