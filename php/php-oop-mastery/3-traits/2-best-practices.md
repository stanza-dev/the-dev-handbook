---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-trait-best-practices"
---

# Trait Best Practices

## Introduction
Traits can be powerful for code reuse, but they can also lead to maintenance nightmares if misused. This lesson covers good use cases, anti-patterns to avoid, and testing strategies for traits.

## Key Concepts
- **Cross-Cutting Concerns**: Functionality like logging, caching, or timestamps that spans multiple unrelated classes.
- **Stateless Utility Methods**: Helper methods that do not depend on instance state.
- **Trait Anti-Patterns**: Using traits as inheritance bypass, hiding dependencies, or trait explosion.
- **Testing Traits**: Using anonymous classes to test trait behavior in isolation.

## Real World Context
You join a project where a `User` class uses 10 traits. Finding where a method is defined requires searching through all 10 trait files. Understanding the class requires reading thousands of lines across multiple files. This is trait explosion, and it makes code unmaintainable.

## Deep Dive

### Good Use Case: Cross-Cutting Concerns

```php
<?php
trait Loggable
{
    protected function log(string $message, string $level = 'info'): void
    {
        $context = [
            'class' => static::class,
            'timestamp' => date('c'),
        ];
        error_log("[$level] $message " . json_encode($context));
    }
}

class UserService
{
    use Loggable;

    public function createUser(array $data): User
    {
        $this->log('Creating user: ' . $data['email']);
        // ...
    }
}

class PaymentService
{
    use Loggable;

    public function processPayment(float $amount): void
    {
        $this->log("Processing payment: $amount");
        // ...
    }
}
```

Logging is a cross-cutting concern that fits naturally into a trait.

### Good Use Case: Stateless Utility Methods

```php
<?php
trait StringHelpers
{
    protected function slugify(string $text): string
    {
        return strtolower(trim(preg_replace('/[^A-Za-z0-9-]+/', '-', $text), '-'));
    }

    protected function truncate(string $text, int $length, string $suffix = '...'): string
    {
        if (strlen($text) <= $length) return $text;
        return substr($text, 0, $length - strlen($suffix)) . $suffix;
    }
}
```

Stateless helpers are safe to use in traits because they have no side effects.

### Anti-Pattern: Traits as Inheritance Bypass

```php
<?php
// BAD: This should be an abstract class
trait BaseEntity
{
    protected int $id;
    protected DateTimeImmutable $createdAt;
    protected DateTimeImmutable $updatedAt;
}

// BETTER: Use abstract class for shared state
abstract class Entity
{
    protected int $id;
    protected DateTimeImmutable $createdAt;
}
```

If a trait defines shared state that every entity needs, it is really an abstract base class in disguise.

### Anti-Pattern: Hidden Dependencies

```php
<?php
// BAD: Trait assumes properties exist
trait Publishable
{
    public function publish(): void
    {
        $this->status = 'published';  // Where does $status come from?
    }
}

// BETTER: Make dependencies explicit
trait Publishable
{
    abstract protected function setStatus(string $status): void;

    public function publish(): void
    {
        $this->setStatus('published');
    }
}
```

Explicit abstract methods document what the trait needs from its host class.

### Testing Traits

```php
<?php
class TimestampableTest extends TestCase
{
    public function testSetsCreatedAt(): void
    {
        $instance = new class {
            use Timestampable;
        };

        $instance->setCreatedAt();

        $this->assertInstanceOf(
            DateTimeImmutable::class,
            $instance->getCreatedAt()
        );
    }
}
```

Anonymous classes let you test traits in isolation without creating dedicated test classes.

## Common Pitfalls
1. **Trait explosion** — Using too many traits makes it impossible to know where methods come from. Limit to 2-3 per class.
2. **Traits replacing DI** — If a trait provides a service (like a mailer), use dependency injection instead.

## Best Practices
1. **Limit traits per class** — If a class uses more than 3-4 traits, refactor some into injected dependencies.
2. **Prefer composition over traits** — Use traits only for truly cross-cutting concerns. For business logic, inject services.

## Summary
- Traits work well for cross-cutting concerns like logging, timestamps, and soft deletes.
- Avoid using traits as inheritance bypass or with hidden dependencies.
- Test traits using anonymous classes.
- Limit traits per class and prefer dependency injection for services.

## Code Examples

**Testing a cacheable trait with anonymous class**

```php
<?php
declare(strict_types=1);

// Testing a trait with anonymous class
trait Cacheable {
    private array $cache = [];

    public function remember(string $key, callable $callback): mixed {
        if (!isset($this->cache[$key])) {
            $this->cache[$key] = $callback();
        }
        return $this->cache[$key];
    }

    public function forget(string $key): void {
        unset($this->cache[$key]);
    }

    public function isCached(string $key): bool {
        return isset($this->cache[$key]);
    }
}

// Test it
$obj = new class { use Cacheable; };

$callCount = 0;
$result = $obj->remember('key', function() use (&$callCount) {
    $callCount++;
    return 'computed';
});

echo $result;  // 'computed'
echo $callCount;  // 1

$obj->remember('key', function() use (&$callCount) {
    $callCount++;
    return 'recomputed';
});
echo $callCount;  // Still 1 - cached!
?>
```


## Resources

- [Traits](https://www.php.net/manual/en/language.oop5.traits.php) — PHP traits documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*