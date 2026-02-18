---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-traits-basics"
---

# Understanding Traits

Traits are a mechanism for code reuse in single inheritance languages. They allow you to share methods across unrelated classes.

## Basic Trait

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
    
    public function getUpdatedAt(): ?DateTimeImmutable {
        return $this->updatedAt;
    }
}

class User {
    use Timestampable;
    
    public function __construct(
        public string $name
    ) {
        $this->setCreatedAt();
    }
}

class Article {
    use Timestampable;
    
    public function __construct(
        public string $title
    ) {
        $this->setCreatedAt();
    }
}
```

## Multiple Traits

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

## Conflict Resolution

```php
<?php
trait A {
    public function hello(): string {
        return 'Hello from A';
    }
}

trait B {
    public function hello(): string {
        return 'Hello from B';
    }
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

## Changing Visibility

```php
<?php
trait Secret {
    private function getSecret(): string {
        return 'secret';
    }
}

class Exposed {
    use Secret {
        getSecret as public;  // Make it public
    }
}

$obj = new Exposed();
echo $obj->getSecret();  // Works - now public
```

## Traits with Abstract Methods

```php
<?php
trait Sluggable {
    public function getSlug(): string {
        return $this->slugify($this->getSlugSource());
    }
    
    private function slugify(string $text): string {
        return strtolower(preg_replace('/[^a-z0-9]+/i', '-', $text));
    }
    
    // Classes using this trait must implement this
    abstract public function getSlugSource(): string;
}

class Article {
    use Sluggable;
    
    public function __construct(
        public string $title
    ) {}
    
    public function getSlugSource(): string {
        return $this->title;
    }
}
```

## Code Examples

**Model traits for soft deletes and UUIDs**

```php
<?php
declare(strict_types=1);

// Common traits for Eloquent-like models
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
    
    public function getDeletedAt(): ?DateTimeImmutable {
        return $this->deletedAt;
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
    
    public function getUuid(): string {
        return $this->uuid;
    }
}

class Document {
    use SoftDeletes, HasUuid;
    
    public function __construct(
        public string $title
    ) {
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