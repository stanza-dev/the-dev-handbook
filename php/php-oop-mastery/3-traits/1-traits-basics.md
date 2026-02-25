---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-traits-basics"
---

# Understanding Traits

## Introduction
Traits are a mechanism for code reuse in single inheritance languages. They allow you to share methods across unrelated classes without inheritance. In PHP 8.5, the `#[\Deprecated]` attribute can now be applied to traits and trait constants.

## Key Concepts
- **Horizontal Reuse**: Traits enable sharing code across classes that do not share a common ancestor.
- **Conflict Resolution**: When two traits define the same method, you must explicitly resolve the conflict.
- **Abstract Methods in Traits**: Traits can require using classes to implement specific methods.
- **Visibility Aliasing**: You can change the visibility of trait methods in the using class.

## Real World Context
In a web application, many unrelated classes need timestamps (createdAt, updatedAt), soft deletes, or UUID generation. These are cross-cutting concerns that do not fit into any inheritance hierarchy. Traits let you add these capabilities to any class.

## Deep Dive

### Basic Trait

```php
<?php
trait Timestampable {
    private ?DateTimeImmutable $createdAt = null;
    private ?DateTimeImmutable $updatedAt = null;

    public function setCreatedAt(): void {
        $this->createdAt = new DateTimeImmutable();
    }

    public function setUpdatedAt(): void {
        $this->updatedAt = new DateTimeImmutable();
    }

    public function getCreatedAt(): ?DateTimeImmutable {
        return $this->createdAt;
    }
}

class User {
    use Timestampable;

    public function __construct(public string $name) {
        $this->setCreatedAt();
    }
}

class Article {
    use Timestampable;

    public function __construct(public string $title) {
        $this->setCreatedAt();
    }
}
```

Both `User` and `Article` gain timestamp methods without any inheritance relationship.

### Multiple Traits

```php
<?php
trait Loggable {
    public function log(string $message): void {
        echo "[LOG] $message\n";
    }
}

trait Serializable {
    public function toArray(): array {
        return get_object_vars($this);
    }

    public function toJson(): string {
        return json_encode($this->toArray());
    }
}

class Product {
    use Loggable, Serializable;

    public function __construct(
        public string $name,
        public float $price
    ) {}
}

$product = new Product('Widget', 9.99);
$product->log('Product created');
echo $product->toJson();
```

A class can use as many traits as it needs, composing behavior from multiple sources.

### Conflict Resolution

When two traits define the same method name, PHP requires you to resolve the conflict explicitly.

```php
<?php
trait A {
    public function hello(): string { return 'Hello from A'; }
}

trait B {
    public function hello(): string { return 'Hello from B'; }
}

class MyClass {
    use A, B {
        A::hello insteadof B;  // Use A's version
        B::hello as helloB;    // Alias B's version
    }
}

$obj = new MyClass();
echo $obj->hello();   // "Hello from A"
echo $obj->helloB();  // "Hello from B"
```

The `insteadof` keyword picks one version, and `as` creates an alias for the other.

### Traits with Abstract Methods

```php
<?php
trait Sluggable {
    public function getSlug(): string {
        return $this->slugify($this->getSlugSource());
    }

    private function slugify(string $text): string {
        return strtolower(preg_replace('/[^a-z0-9]+/i', '-', $text));
    }

    abstract public function getSlugSource(): string;
}

class Article {
    use Sluggable;

    public function __construct(public string $title) {}

    public function getSlugSource(): string {
        return $this->title;
    }
}
```

Abstract methods in traits make dependencies explicit rather than hidden.

### PHP 8.5: Deprecated Attribute on Traits

In PHP 8.5, you can mark a trait as deprecated using the `#[\Deprecated]` attribute.

```php
<?php
#[\Deprecated('Use LoggerInterface injection instead', since: '2.0')]
trait Loggable {
    public function log(string $message): void {
        echo "[LOG] $message\n";
    }
}

// Using this trait will trigger a deprecation notice
class MyService {
    use Loggable; // Deprecation notice at compile time
}
```

This helps library authors migrate users away from traits toward better patterns like dependency injection.

## Common Pitfalls
1. **Hidden dependencies** — Traits that assume properties exist without declaring them create fragile code. Use abstract methods instead.
2. **Trait explosion** — Using 8+ traits on a class makes it impossible to understand where methods come from.

## Best Practices
1. **Keep traits stateless when possible** — Traits that only add behavior (not state) are easier to reason about.
2. **Use abstract methods for dependencies** — If a trait needs data from the class, declare an abstract method.

## Summary
- Traits enable horizontal code reuse across unrelated classes.
- Use `insteadof` and `as` to resolve method conflicts between traits.
- Abstract methods in traits make dependencies explicit.
- PHP 8.5 allows `#[\Deprecated]` on traits and trait constants.

## Code Examples

**Model traits for soft deletes and UUIDs**

```php
<?php
declare(strict_types=1);

trait SoftDeletes {
    private ?DateTimeImmutable $deletedAt = null;

    public function delete(): void {
        $this->deletedAt = new DateTimeImmutable();
    }

    public function restore(): void {
        $this->deletedAt = null;
    }

    public function isDeleted(): bool {
        return $this->deletedAt !== null;
    }
}

trait HasUuid {
    private string $uuid;

    public function initializeUuid(): void {
        $this->uuid = sprintf(
            '%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
            mt_rand(0, 0xffff), mt_rand(0, 0xffff),
            mt_rand(0, 0xffff),
            mt_rand(0, 0x0fff) | 0x4000,
            mt_rand(0, 0x3fff) | 0x8000,
            mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff)
        );
    }

    public function getUuid(): string { return $this->uuid; }
}

class Document {
    use SoftDeletes, HasUuid;

    public function __construct(public string $title) {
        $this->initializeUuid();
    }
}

$doc = new Document('My Document');
echo $doc->getUuid() . "\n";
$doc->delete();
echo $doc->isDeleted() ? 'Deleted' : 'Active';
?>
```


## Resources

- [Traits](https://www.php.net/manual/en/language.oop5.traits.php) — PHP traits documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*