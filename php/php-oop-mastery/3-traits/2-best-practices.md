---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-trait-best-practices"
---

# Trait Best Practices

Traits can be powerful for code reuse, but they can also lead to maintenance nightmares if misused. Follow these best practices.

## Good Use Cases

### 1. Cross-Cutting Concerns

```php
<?php
// Logging concern shared across unrelated classes
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
    
    protected function logError(Throwable $e): void
    {
        $this->log($e->getMessage(), 'error');
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

### 2. Stateless Utility Methods

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
        if (strlen($text) <= $length) {
            return $text;
        }
        return substr($text, 0, $length - strlen($suffix)) . $suffix;
    }
}
```

## Anti-Patterns to Avoid

### 1. Traits as Inheritance Bypass

```php
<?php
// BAD: Using trait to share state that should be inherited
trait BaseEntity
{
    protected int $id;
    protected DateTimeImmutable $createdAt;
    protected DateTimeImmutable $updatedAt;
    
    // This should probably be an abstract class!
}

// BETTER: Use abstract class for shared state
abstract class Entity
{
    protected int $id;
    protected DateTimeImmutable $createdAt;
    protected DateTimeImmutable $updatedAt;
}
```

### 2. Traits with Hidden Dependencies

```php
<?php
// BAD: Trait assumes properties exist
trait Publishable
{
    public function publish(): void
    {
        $this->status = 'published';  // Where does $status come from?
        $this->publishedAt = new DateTime();  // Magic!
    }
}

// BETTER: Make dependencies explicit
trait Publishable
{
    abstract protected function getStatus(): string;
    abstract protected function setStatus(string $status): void;
    abstract protected function setPublishedAt(DateTimeInterface $date): void;
    
    public function publish(): void
    {
        $this->setStatus('published');
        $this->setPublishedAt(new DateTimeImmutable());
    }
}
```

### 3. Trait Explosion

```php
<?php
// BAD: Too many traits make code hard to follow
class User
{
    use Timestamps;
    use SoftDeletes;
    use HasUuid;
    use Searchable;
    use Cacheable;
    use Loggable;
    use Validatable;
    use Serializable;
    // Where does any method come from?!
}

// BETTER: Limit traits, prefer composition
class User
{
    use Timestamps;  // Just timestamps
    
    public function __construct(
        private CacheInterface $cache,
        private LoggerInterface $logger
    ) {}
}
```

## Testing Traits

```php
<?php
// Create anonymous class to test trait
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

## Resources

- [Traits](https://www.php.net/manual/en/language.oop5.traits.php) — PHP traits documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*