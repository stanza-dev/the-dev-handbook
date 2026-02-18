---
source_course: "php-modern-features"
source_lesson: "php-modern-features-lazy-objects"
---

# Lazy Objects (PHP 8.4)

Lazy objects defer initialization until actually needed. PHP 8.4 introduces native lazy object support through the Reflection API.

## Why Lazy Loading?

```php
<?php
// Without lazy loading:
// All services are instantiated even if unused
class Container
{
    public function __construct()
    {
        $this->db = new DatabaseConnection();      // Heavy!
        $this->cache = new RedisConnection();      // Heavy!
        $this->mailer = new SmtpMailer();          // Heavy!
    }
}

// With lazy loading:
// Services only created when accessed
```

## Native Lazy Objects (PHP 8.4)

```php
<?php
class ExpensiveService
{
    public function __construct()
    {
        echo "Expensive initialization...\n";
        sleep(2);  // Simulate heavy setup
    }
    
    public function doWork(): string
    {
        return "Work done!";
    }
}

// Create lazy ghost object
$reflector = new ReflectionClass(ExpensiveService::class);

$lazy = $reflector->newLazyGhost(function(ExpensiveService $object) {
    // This initializer runs only when object is first accessed
    $object->__construct();
});

echo "Lazy object created\n";  // No initialization yet!
echo $lazy->doWork();          // NOW initialization happens
```

## Lazy Proxy Pattern

```php
<?php
class DatabaseConnection
{
    private PDO $pdo;
    
    public function __construct(string $dsn)
    {
        echo "Connecting to database...\n";
        $this->pdo = new PDO($dsn);
    }
    
    public function query(string $sql): array
    {
        return $this->pdo->query($sql)->fetchAll();
    }
}

// Create lazy proxy
$reflector = new ReflectionClass(DatabaseConnection::class);

$proxy = $reflector->newLazyProxy(function() {
    // Factory returns the real object
    return new DatabaseConnection('mysql:host=localhost');
});

// Connection not made yet!
echo "Proxy created\n";

// First method call triggers initialization
$result = $proxy->query('SELECT 1');
```

## Ghost vs Proxy

| Feature | Lazy Ghost | Lazy Proxy |
|---------|------------|------------|
| Identity | Same object | Wraps real object |
| `===` check | Works normally | Different identity |
| Use case | Most cases | When identity matters |

## Checking Lazy State

```php
<?php
$reflector = new ReflectionClass($object);

// Check if object is lazy and uninitialized
if ($reflector->isUninitializedLazyObject($object)) {
    echo "Object not yet initialized\n";
}

// Force initialization
$reflector->initializeLazyObject($object);

// Reset to uninitialized state
$reflector->resetAsLazyGhost($object, function($obj) {
    $obj->__construct();
});
```

## Practical Container Example

```php
<?php
class LazyContainer
{
    private array $factories = [];
    private array $instances = [];
    
    public function register(string $id, callable $factory): void
    {
        $this->factories[$id] = $factory;
    }
    
    public function get(string $id): object
    {
        if (!isset($this->instances[$id])) {
            $reflector = new ReflectionClass($id);
            
            $this->instances[$id] = $reflector->newLazyGhost(
                function($object) use ($id) {
                    $real = ($this->factories[$id])();
                    
                    // Copy properties from real to ghost
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
```

## Resources

- [Lazy Objects RFC](https://wiki.php.net/rfc/lazy-objects) — PHP 8.4 lazy objects RFC

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*