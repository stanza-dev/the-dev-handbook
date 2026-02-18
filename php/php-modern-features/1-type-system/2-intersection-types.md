---
source_course: "php-modern-features"
source_lesson: "php-modern-features-intersection-types"
---

# Intersection Types (PHP 8.1+)

Intersection types require a value to satisfy multiple type constraints simultaneously. A value must implement ALL specified interfaces.

## Basic Syntax

```php
<?php
interface Countable {
    public function count(): int;
}

interface Traversable {}

// Value must implement BOTH interfaces
function process(Countable&Traversable $collection): void {
    foreach ($collection as $item) {
        // Can iterate
    }
    echo "Count: " . $collection->count();
}
```

## Use Case: Combining Interfaces

```php
<?php
interface Renderable {
    public function render(): string;
}

interface Cacheable {
    public function getCacheKey(): string;
}

// Must implement both interfaces
function renderAndCache(Renderable&Cacheable $component): string {
    $key = $component->getCacheKey();
    
    // Check cache first...
    
    return $component->render();
}
```

## Intersection vs Union

```php
<?php
// Union: A OR B (either one)
function union(A|B $value): void {}

// Intersection: A AND B (both required)
function intersection(A&B $value): void {}
```

## DNF Types (PHP 8.2+)

Disjunctive Normal Form combines union and intersection:

```php
<?php
// (A&B)|C means: (both A and B) OR just C
function process((Countable&Iterator)|null $data): void {
    if ($data === null) {
        return;
    }
    
    // $data implements both Countable and Iterator
}

// More complex example
function handle((A&B)|(C&D) $value): void {
    // Value is either (A and B) or (C and D)
}
```

## Practical Example

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
    // Guaranteed to have both methods
    $context = $item->getLogContext();
    $json = $item->toJson();
}
```

## Resources

- [Intersection Types](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.composite.intersection) — Official documentation for intersection types

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*