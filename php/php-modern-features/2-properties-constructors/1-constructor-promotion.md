---
source_course: "php-modern-features"
source_lesson: "php-modern-features-constructor-promotion"
---

# Constructor Property Promotion (PHP 8.0+)

Constructor promotion is a shorthand syntax that declares and initializes properties directly in the constructor signature.

## The Old Way

```php
<?php
// Before PHP 8.0
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

## The New Way

```php
<?php
// PHP 8.0+ with promotion
class User {
    public function __construct(
        private string $name,
        private string $email,
        private int $age
    ) {}
}

// That's it! Properties are declared and assigned automatically
```

## Mixed Promotion and Regular Parameters

```php
<?php
class Post {
    public function __construct(
        private string $title,      // Promoted
        private string $content,    // Promoted
        private Author $author,     // Promoted
        string $tempData = ''       // NOT promoted (no visibility)
    ) {
        // $tempData is available here but not as a property
        $this->processTemp($tempData);
    }
}
```

## With Default Values

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

## Combining with Readonly (PHP 8.1+)

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

## Benefits

1. **Less boilerplate**: No property declarations, no assignments
2. **Cleaner code**: Everything in one place
3. **Easier maintenance**: Change once, not three times
4. **Works with named arguments**: `new User(email: 'test@test.com', name: 'Test')`

## Code Examples

**Data Transfer Object with constructor promotion**

```php
<?php
declare(strict_types=1);

// Modern DTO with constructor promotion
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

// Usage
$dto = CreateUserDTO::fromRequest($_POST);
echo $dto->name;
?>
```


## Resources

- [Constructor Promotion](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion) — Official documentation for constructor property promotion

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*