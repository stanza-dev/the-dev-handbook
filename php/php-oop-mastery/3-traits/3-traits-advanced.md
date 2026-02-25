---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-traits-advanced"
---

# Advanced Trait Patterns

## Introduction
Beyond basic code sharing, traits support advanced patterns like composing multiple behaviors, enforcing contracts via abstract methods, and integrating with PHP's attribute system. This lesson covers these advanced uses.

## Key Concepts
- **Trait Composition**: Building complex behaviors from multiple small traits.
- **Trait Constants**: Shared constants within traits (PHP 8.2+).
- **Attribute Integration**: Using PHP attributes on trait methods for metadata.
- **Trait + Interface Pattern**: Combining interfaces for contracts with traits for default implementations.

## Real World Context
Frameworks like Laravel and Symfony use advanced trait patterns extensively. Laravel's Eloquent models compose behaviors like `SoftDeletes`, `HasFactory`, `HasTimestamps` through traits. Understanding advanced patterns helps you build your own reusable components.

## Deep Dive

### Trait + Interface Pattern

The most robust pattern combines an interface (contract) with a trait (default implementation).

```php
<?php
interface HasTimestamps {
    public function getCreatedAt(): ?DateTimeImmutable;
    public function getUpdatedAt(): ?DateTimeImmutable;
    public function touch(): void;
}

trait Timestampable {
    private ?DateTimeImmutable $createdAt = null;
    private ?DateTimeImmutable $updatedAt = null;

    public function getCreatedAt(): ?DateTimeImmutable {
        return $this->createdAt;
    }

    public function getUpdatedAt(): ?DateTimeImmutable {
        return $this->updatedAt;
    }

    public function touch(): void {
        if ($this->createdAt === null) {
            $this->createdAt = new DateTimeImmutable();
        }
        $this->updatedAt = new DateTimeImmutable();
    }
}

class Article implements HasTimestamps {
    use Timestampable;

    public function __construct(public string $title) {
        $this->touch();
    }
}

function logEntity(HasTimestamps $entity): void {
    echo "Created: " . $entity->getCreatedAt()?->format('Y-m-d') . "\n";
}
```

The interface enables type-hinting, while the trait provides the implementation. Classes can override specific methods if they need custom behavior.

### Trait Constants (PHP 8.2+)

```php
<?php
trait HttpStatus {
    public const OK = 200;
    public const NOT_FOUND = 404;
    public const SERVER_ERROR = 500;
}

class Response {
    use HttpStatus;

    public function __construct(
        private int $status = self::OK
    ) {}
}

echo Response::OK; // 200
```

Trait constants let you share named values alongside methods. In PHP 8.5, these constants can also be marked with `#[\Deprecated]`.

### Composing Multiple Traits

```php
<?php
trait HasUuid {
    private string $uuid;
    public function initUuid(): void {
        $this->uuid = bin2hex(random_bytes(16));
    }
    public function getUuid(): string { return $this->uuid; }
}

trait SoftDeletes {
    private ?DateTimeImmutable $deletedAt = null;
    public function softDelete(): void {
        $this->deletedAt = new DateTimeImmutable();
    }
    public function isDeleted(): bool {
        return $this->deletedAt !== null;
    }
    public function restore(): void {
        $this->deletedAt = null;
    }
}

trait Auditable {
    private string $lastModifiedBy = '';
    public function setModifiedBy(string $user): void {
        $this->lastModifiedBy = $user;
    }
    public function getModifiedBy(): string {
        return $this->lastModifiedBy;
    }
}

class Document {
    use HasUuid, SoftDeletes, Auditable, Timestampable;

    public function __construct(public string $title) {
        $this->initUuid();
        $this->touch();
    }
}
```

Each trait adds a single, focused capability. Together they compose a feature-rich class.

### Trait Precedence Rules

PHP resolves methods in this order: class methods override trait methods, which override inherited methods.

```php
<?php
trait Greetable {
    public function greet(): string {
        return 'Hello from trait';
    }
}

class Base {
    public function greet(): string {
        return 'Hello from base';
    }
}

class Child extends Base {
    use Greetable; // Trait wins over Base
}

class Override extends Base {
    use Greetable;

    public function greet(): string {
        return 'Hello from class'; // Class wins over trait
    }
}
```

Remember: Class > Trait > Parent Class.

## Common Pitfalls
1. **Ignoring precedence rules** — Traits override parent class methods, which can be surprising. Always be aware of the resolution order.
2. **State conflicts** — Two traits defining the same property name cause a fatal error that cannot be resolved with `insteadof`.

## Best Practices
1. **Pair traits with interfaces** — This gives you both type safety (interface) and convenience (default implementation).
2. **Prefix trait properties** — Use descriptive names to avoid collisions when multiple traits are composed.

## Summary
- The Trait + Interface pattern provides both contracts and default implementations.
- Trait constants (PHP 8.2+) share values alongside methods.
- Precedence: Class > Trait > Parent Class.
- PHP 8.5 adds `#[\Deprecated]` support for traits and trait constants.

## Code Examples

**Trait + Interface pattern for publishable content**

```php
<?php
declare(strict_types=1);

interface Publishable {
    public function publish(): void;
    public function unpublish(): void;
    public function isPublished(): bool;
}

trait PublishableTrait {
    private bool $published = false;
    private ?DateTimeImmutable $publishedAt = null;

    public function publish(): void {
        $this->published = true;
        $this->publishedAt = new DateTimeImmutable();
    }

    public function unpublish(): void {
        $this->published = false;
        $this->publishedAt = null;
    }

    public function isPublished(): bool {
        return $this->published;
    }

    public function getPublishedAt(): ?DateTimeImmutable {
        return $this->publishedAt;
    }
}

class BlogPost implements Publishable {
    use PublishableTrait;

    public function __construct(
        public readonly string $title,
        public readonly string $content
    ) {}
}

$post = new BlogPost('Hello World', 'Content here');
$post->publish();
echo $post->isPublished() ? 'Published' : 'Draft';
?>
```


## Resources

- [Traits](https://www.php.net/manual/en/language.oop5.traits.php) — PHP traits documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*