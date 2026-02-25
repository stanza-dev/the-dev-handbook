---
source_course: "php-modern-features"
source_lesson: "php-modern-features-lazy-objects"
---

# Lazy Objects

## Introduction
Lazy objects, introduced in PHP 8.4, defer their initialization until they are actually used. This is a native solution to a problem previously solved only by userland proxies and code generation, such as in Doctrine ORM's entity proxies.

## Key Concepts
- **Lazy Ghost**: A lazy object that initializes itself in-place when first accessed. The object identity stays the same.
- **Lazy Proxy**: A wrapper that creates and delegates to a real object on first access. The identity differs from the real object.
- **Deferred Initialization**: The pattern of delaying expensive setup (database connections, file reads) until the resource is actually needed.

## Real World Context
Dependency injection containers often instantiate dozens of services at startup, even though a single request may only use a few. Lazy objects solve this by deferring expensive initialization until the service is actually called, reducing memory usage and startup time.

## Deep Dive

### Why Lazy Loading?

Consider a container that instantiates everything eagerly:

```php
<?php
class Container {
    public function __construct() {
        $this->db = new DatabaseConnection();      // Heavy!
        $this->cache = new RedisConnection();      // Heavy!
        $this->mailer = new SmtpMailer();          // Heavy!
    }
}
```

All three connections are established even if the request only needs the database. Lazy loading defers creation to first use.

### Native Lazy Ghost Objects (PHP 8.4)

Create a lazy ghost using the Reflection API:

```php
<?php
class ExpensiveService {
    public function __construct() {
        echo "Expensive initialization...\n";
        sleep(2);
    }
    
    public function doWork(): string {
        return "Work done!";
    }
}

$reflector = new ReflectionClass(ExpensiveService::class);

$lazy = $reflector->newLazyGhost(function(ExpensiveService $object) {
    $object->__construct();
});

echo "Lazy object created\n";  // No initialization yet!
echo $lazy->doWork();          // NOW initialization happens
```

The initializer closure runs only when the object is first accessed. Until then, no resources are consumed.

### Lazy Proxy Pattern

Proxies delegate to a separate real object:

```php
<?php
$reflector = new ReflectionClass(DatabaseConnection::class);

$proxy = $reflector->newLazyProxy(function() {
    return new DatabaseConnection('mysql:host=localhost');
});

echo "Proxy created\n";  // Connection not made yet!
$result = $proxy->query('SELECT 1');  // First call triggers creation
```

The factory closure returns the real object, which the proxy then delegates to.

### Ghost vs Proxy Comparison

| Feature | Lazy Ghost | Lazy Proxy |
|---------|------------|------------|
| Identity | Same object | Wraps real object |
| `===` check | Works normally | Different identity |
| Use case | Most cases | When factory returns existing instance |

### Checking and Controlling Lazy State

You can inspect and control lazy objects:

```php
<?php
$reflector = new ReflectionClass($object);

if ($reflector->isUninitializedLazyObject($object)) {
    echo "Object not yet initialized\n";
}

$reflector->initializeLazyObject($object);  // Force init
```

This is useful for debugging, preloading, and testing.

## Common Pitfalls
1. **Assuming lazy objects are free** — Lazy objects have a small overhead per access for the initialization check. For objects accessed thousands of times in a tight loop, eager initialization may be better.
2. **Identity issues with proxies** — Lazy proxies have a different identity from the real object, so `$proxy === $realObject` is false. Use lazy ghosts when identity matters.

## Best Practices
1. **Use lazy ghosts by default** — They preserve object identity and work with most use cases. Only use proxies when the factory must return an existing instance.
2. **Prefer DI container integration** — Let your dependency injection container handle lazy loading rather than manually creating lazy objects in business code.

## Summary
- PHP 8.4 provides native lazy object support via the Reflection API.
- Lazy ghosts initialize in-place and preserve object identity.
- Lazy proxies delegate to a factory-created real object.
- Lazy objects reduce startup cost by deferring expensive initialization.
- Use `isUninitializedLazyObject()` to check if initialization has occurred.

## Code Examples

**A simple DI container using lazy ghost objects to defer service initialization until first use**

```php
<?php
declare(strict_types=1);

class LazyContainer {
    private array $factories = [];
    private array $instances = [];
    
    public function register(string $id, callable $factory): void {
        $this->factories[$id] = $factory;
    }
    
    public function get(string $id): object {
        if (!isset($this->instances[$id])) {
            $reflector = new ReflectionClass($id);
            
            $this->instances[$id] = $reflector->newLazyGhost(
                function($object) use ($id) {
                    $real = ($this->factories[$id])();
                    $r = new ReflectionObject($real);
                    foreach ($r->getProperties() as $prop) {
                        $prop->setAccessible(true);
                        $prop->setValue($object, $prop->getValue($real));
                    }
                }
            );
        }
        return $this->instances[$id];
    }
}

// Services are created lazily on first use
$container = new LazyContainer();
$container->register(DatabaseConnection::class, fn() => new DatabaseConnection('mysql:host=localhost'));
$db = $container->get(DatabaseConnection::class);  // Not initialized yet
$db->query('SELECT 1');  // NOW it initializes
?>
```


## Resources

- [Lazy Objects RFC](https://wiki.php.net/rfc/lazy-objects) — PHP 8.4 lazy objects RFC

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*