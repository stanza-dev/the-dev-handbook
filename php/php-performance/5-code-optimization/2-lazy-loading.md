---
source_course: "php-performance"
source_lesson: "php-performance-lazy-loading"
---

# Lazy Loading & Deferred Execution

## Introduction
Don't do work until you need the results.

## Key Concepts
- **Lazy Loading**: Deferring expensive initialization until the resource is actually needed.
- **Lazy Objects (PHP 8.4+)**: `ReflectionClass::newLazyProxy()` creates objects whose initialization is deferred until first property access.
- **Service Container Lazy Loading**: DI containers like Symfony's create service proxies that only instantiate the real service when a method is called.
- **Autoloading**: PSR-4 autoloading ensures PHP only loads class files when they're first referenced, not at startup.

## Real World Context
Symfony's dependency injection container uses lazy loading extensively — services that aren't used during a request are never instantiated. This can save 50-100ms per request in large applications with hundreds of services. PHP 8.4's native lazy objects make this pattern available without framework-specific proxies.

## Deep Dive
### Intro

Don't do work until you need the results.

### Lazy object initialization

```php
<?php
class ServiceContainer
{
    private array $services = [];
    private array $factories = [];
    
    public function register(string $name, callable $factory): void
    {
        $this->factories[$name] = $factory;
    }
    
    public function get(string $name): object
    {
        // Only instantiate when requested
        if (!isset($this->services[$name])) {
            $this->services[$name] = ($this->factories[$name])();
        }
        
        return $this->services[$name];
    }
}

$container = new ServiceContainer();

// Register factories (cheap)
$container->register('database', fn() => new PDO(...));
$container->register('mailer', fn() => new Mailer(...));
$container->register('logger', fn() => new Logger(...));

// Only mailer is actually instantiated
$mailer = $container->get('mailer');
```

### Generators for lazy iteration

```php
<?php
// Eager: Loads all users into memory
function getAllUsers(): array {
    return $pdo->query('SELECT * FROM users')->fetchAll();
}

// Lazy: One user at a time
function getAllUsers(): Generator {
    $stmt = $pdo->query('SELECT * FROM users');
    while ($row = $stmt->fetch()) {
        yield new User($row);
    }
}

// Process million users with minimal memory
foreach (getAllUsers() as $user) {
    processUser($user);
}
```

### Lazy collections

```php
<?php
class LazyCollection implements IteratorAggregate
{
    private array $operations = [];
    
    public function __construct(
        private iterable $source
    ) {}
    
    public function map(callable $fn): self
    {
        $this->operations[] = ['map', $fn];
        return $this;
    }
    
    public function filter(callable $fn): self
    {
        $this->operations[] = ['filter', $fn];
        return $this;
    }
    
    public function getIterator(): Generator
    {
        foreach ($this->source as $item) {
            $include = true;
            
            foreach ($this->operations as [$op, $fn]) {
                if ($op === 'map') {
                    $item = $fn($item);
                } elseif ($op === 'filter') {
                    if (!$fn($item)) {
                        $include = false;
                        break;
                    }
                }
            }
            
            if ($include) {
                yield $item;
            }
        }
    }
    
    // Only here do we actually iterate
    public function toArray(): array
    {
        return iterator_to_array($this);
    }
}

// Operations are chained but not executed
$result = (new LazyCollection($hugeDataset))
    ->filter(fn($x) => $x > 0)
    ->map(fn($x) => $x * 2)
    ->filter(fn($x) => $x < 100);

// Only now does processing happen
foreach ($result as $item) {
    echo $item;
}
```

### Deferred calculations

```php
<?php
class Report
{
    private ?array $data = null;
    
    // Expensive calculation deferred
    public function getData(): array
    {
        if ($this->data === null) {
            $this->data = $this->calculateExpensiveReport();
        }
        return $this->data;
    }
    
    // User might never need this
    private function calculateExpensiveReport(): array
    {
        // Complex queries and calculations
        return [...];
    }
}
```

## Common Pitfalls
1. **Over-using lazy loading** — For lightweight objects used on every request, lazy loading adds overhead without benefit. Only apply it to expensive initializations (database connections, API clients).
2. **Not considering initialization timing** — A lazy-loaded service that fails during initialization will throw an exception at an unexpected point in your code. Handle initialization errors gracefully.

## Best Practices
1. **Use lazy loading for heavy services** — Database connections, HTTP clients, file system wrappers, and cache clients are ideal candidates for lazy initialization.
2. **Leverage PHP 8.4+ lazy objects** — Use `ReflectionClass::newLazyProxy()` for framework-independent lazy loading without external dependencies.

## Summary
- Lazy loading defers expensive object initialization until the resource is actually needed.
- PHP 8.4+ provides native lazy objects via `ReflectionClass::newLazyProxy()`.
- Apply lazy loading to heavy services (DB connections, API clients) but not to lightweight objects used on every request.

## Code Examples

**Lazy pipeline for memory-efficient data processing**

```php
<?php
declare(strict_types=1);

// Complete lazy pipeline implementation
class Pipeline
{
    /** @var callable[] */
    private array $stages = [];
    
    public static function from(iterable $source): self
    {
        $pipeline = new self();
        $pipeline->stages[] = fn() => yield from $source;
        return $pipeline;
    }
    
    public function map(callable $fn): self
    {
        $prev = $this->stages;
        $this->stages[] = function() use ($prev, $fn) {
            foreach ($this->execute($prev) as $item) {
                yield $fn($item);
            }
        };
        return $this;
    }
    
    public function filter(callable $fn): self
    {
        $prev = $this->stages;
        $this->stages[] = function() use ($prev, $fn) {
            foreach ($this->execute($prev) as $item) {
                if ($fn($item)) {
                    yield $item;
                }
            }
        };
        return $this;
    }
    
    public function take(int $n): self
    {
        $prev = $this->stages;
        $this->stages[] = function() use ($prev, $n) {
            $count = 0;
            foreach ($this->execute($prev) as $item) {
                if ($count >= $n) break;
                yield $item;
                $count++;
            }
        };
        return $this;
    }
    
    public function chunk(int $size): self
    {
        $prev = $this->stages;
        $this->stages[] = function() use ($prev, $size) {
            $chunk = [];
            foreach ($this->execute($prev) as $item) {
                $chunk[] = $item;
                if (count($chunk) === $size) {
                    yield $chunk;
                    $chunk = [];
                }
            }
            if ($chunk) yield $chunk;
        };
        return $this;
    }
    
    private function execute(array $stages): Generator
    {
        $current = null;
        foreach ($stages as $stage) {
            $current = $stage();
        }
        yield from $current ?? [];
    }
    
    public function toArray(): array
    {
        return iterator_to_array($this->run());
    }
    
    public function run(): Generator
    {
        yield from $this->execute($this->stages);
    }
    
    public function each(callable $fn): void
    {
        foreach ($this->run() as $item) {
            $fn($item);
        }
    }
    
    public function reduce(callable $fn, mixed $initial = null): mixed
    {
        $result = $initial;
        foreach ($this->run() as $item) {
            $result = $fn($result, $item);
        }
        return $result;
    }
}

// Usage: Process million records with minimal memory
$total = Pipeline::from(readLargeFile('sales.csv'))
    ->map(fn($line) => str_getcsv($line))
    ->filter(fn($row) => $row[2] === 'completed')
    ->map(fn($row) => (float) $row[3])
    ->reduce(fn($sum, $amount) => $sum + $amount, 0.0);

echo "Total sales: $total";
?>
```


## Resources

- [Generators](https://www.php.net/manual/en/language.generators.php) — PHP generators for memory-efficient iteration

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*