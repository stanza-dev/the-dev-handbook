---
source_course: "php-modern-features"
source_lesson: "php-modern-features-intersection-types"
---

# Intersection Types

## Introduction
Intersection types, introduced in PHP 8.1, require a value to satisfy multiple type constraints simultaneously. While union types say "A or B", intersection types say "A and B" — the value must implement all specified interfaces.

## Key Concepts
- **Intersection Type**: A type declaration using `&` that requires all listed types, e.g. `Countable&Traversable`.
- **DNF Types (PHP 8.2+)**: Disjunctive Normal Form allows combining union and intersection types, e.g. `(A&B)|null`.

## Real World Context
Intersection types solve a real problem: you want to accept an object that implements multiple interfaces without creating a new interface just for that combination. This is common in service layers where a dependency must be both loggable and serializable, or both countable and iterable.

## Deep Dive

### Basic Syntax

Intersection types use the ampersand `&` between types:

```php
<?php
// Countable and Iterator are built-in PHP interfaces
function process(Countable&Iterator $collection): void {
    foreach ($collection as $item) {
        // Can iterate (Iterator)
    }
    echo "Count: " . $collection->count();  // Can count (Countable)
}
```

The parameter `$collection` must implement both `Countable` and `Iterator`. Passing an object that only implements one of them would cause a `TypeError`.

### Combining Interfaces

This pattern shines when you need to guarantee multiple capabilities:

```php
<?php
interface Renderable {
    public function render(): string;
}

interface Cacheable {
    public function getCacheKey(): string;
}

function renderAndCache(Renderable&Cacheable $component): string {
    $key = $component->getCacheKey();
    return $component->render();
}
```

Both methods are guaranteed to exist on `$component`, with no need for `instanceof` checks.

### Intersection vs Union

The key difference is logical AND vs OR:

```php
<?php
// Union: value implements A OR B (either one suffices)
function union(A|B $value): void {}

// Intersection: value implements A AND B (both required)
function intersection(A&B $value): void {}
```

Union types are more permissive; intersection types are more restrictive.

### DNF Types (PHP 8.2+)

Disjunctive Normal Form combines union and intersection types in a structured way:

```php
<?php
// (A&B)|C means: (both A and B) OR just C
function process((Countable&Iterator)|null $data): void {
    if ($data === null) {
        return;
    }
    // $data implements both Countable and Iterator
}

// More complex DNF
function handle((A&B)|(C&D) $value): void {
    // Value is either (A and B) or (C and D)
}
```

DNF types must follow a specific structure: intersections grouped in parentheses, combined with unions at the top level.

### Practical Example

Here is a complete example showing intersection types in a real scenario:

```php
<?php
interface Loggable {
    public function getLogContext(): array;
}

interface Jsonable {
    public function toJson(): string;
}

class Event implements Loggable, Jsonable {
    public function __construct(
        public string $type,
        public array $data
    ) {}
    
    public function getLogContext(): array {
        return ['type' => $this->type, 'data' => $this->data];
    }
    
    public function toJson(): string {
        return json_encode($this->data);
    }
}

function logAndSerialize(Loggable&Jsonable $item): void {
    $context = $item->getLogContext();
    $json = $item->toJson();
}
```

The `logAndSerialize` function can safely call methods from both interfaces without any runtime checks.

## Common Pitfalls
1. **Using intersection with scalar types** — Intersection types only work with class/interface types. You cannot write `int&string` because no value can be both simultaneously.
2. **Forgetting parentheses in DNF** — `(A&B)|null` is valid DNF, but `A&B|null` is a syntax error. Always wrap intersection groups in parentheses when combining with unions.

## Best Practices
1. **Use intersection types to avoid marker interfaces** — Instead of creating `interface RenderableAndCacheable extends Renderable, Cacheable {}`, use `Renderable&Cacheable` directly in the type declaration.
2. **Combine with nullable for optional parameters** — `(Countable&Iterator)|null` cleanly expresses an optional parameter that, when provided, must satisfy both constraints.

## Summary
- Intersection types use `&` to require all listed interfaces simultaneously.
- They only work with class and interface types, not scalars.
- DNF types (PHP 8.2) combine unions and intersections: `(A&B)|C`.
- They eliminate the need for combined marker interfaces.
- Use parentheses when mixing intersection and union types.

## Code Examples

**Using intersection types to guarantee both string and JSON serialization capabilities on a single parameter**

```php
<?php
declare(strict_types=1);

// Stringable and JsonSerializable are built-in PHP interfaces
// No need to declare them — just implement them

// Intersection type ensures both serialization capabilities
function export(Stringable&JsonSerializable $item): array {
    return [
        'text' => (string) $item,
        'json' => json_encode($item->jsonSerialize()),
    ];
}

class Product implements Stringable, JsonSerializable {
    public function __construct(
        public string $name,
        public float $price
    ) {}
    
    public function __toString(): string {
        return "{$this->name} (\${$this->price})";
    }
    
    public function jsonSerialize(): mixed {
        return ['name' => $this->name, 'price' => $this->price];
    }
}

$product = new Product('Widget', 29.99);
$exported = export($product);
// ['text' => 'Widget ($29.99)', 'json' => '{"name":"Widget","price":29.99}']
?>
```


## Resources

- [Intersection Types](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.composite.intersection) — Official documentation for intersection types

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*