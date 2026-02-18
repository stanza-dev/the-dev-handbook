---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-di-container"
---

# Building a Simple DI Container

A DI Container manages object creation and dependency resolution automatically.

## Basic Container

```php
<?php
class Container {
    private array $bindings = [];
    private array $instances = [];
    
    // Bind an interface to a concrete class
    public function bind(string $abstract, string|callable $concrete): void {
        $this->bindings[$abstract] = $concrete;
    }
    
    // Bind a singleton
    public function singleton(string $abstract, string|callable $concrete): void {
        $this->bind($abstract, function($container) use ($abstract, $concrete) {
            if (!isset($this->instances[$abstract])) {
                $this->instances[$abstract] = is_callable($concrete) 
                    ? $concrete($container) 
                    : $container->make($concrete);
            }
            return $this->instances[$abstract];
        });
    }
    
    // Resolve a class
    public function make(string $abstract): object {
        // Check if we have a binding
        if (isset($this->bindings[$abstract])) {
            $concrete = $this->bindings[$abstract];
            
            if (is_callable($concrete)) {
                return $concrete($this);
            }
            
            return $this->make($concrete);
        }
        
        // Auto-resolve using reflection
        return $this->resolve($abstract);
    }
    
    private function resolve(string $class): object {
        $reflection = new ReflectionClass($class);
        
        if (!$reflection->isInstantiable()) {
            throw new Exception("Cannot instantiate $class");
        }
        
        $constructor = $reflection->getConstructor();
        
        if ($constructor === null) {
            return new $class();
        }
        
        $dependencies = [];
        
        foreach ($constructor->getParameters() as $param) {
            $type = $param->getType();
            
            if ($type === null || $type->isBuiltin()) {
                if ($param->isDefaultValueAvailable()) {
                    $dependencies[] = $param->getDefaultValue();
                } else {
                    throw new Exception("Cannot resolve param: {$param->getName()}");
                }
            } else {
                $dependencies[] = $this->make($type->getName());
            }
        }
        
        return $reflection->newInstanceArgs($dependencies);
    }
}
```

## Using the Container

```php
<?php
$container = new Container();

// Bind interfaces to implementations
$container->bind(UserRepositoryInterface::class, MySQLUserRepository::class);
$container->bind(MailerInterface::class, SmtpMailer::class);

// Bind with factory function
$container->singleton(PDO::class, function() {
    return new PDO(
        'mysql:host=localhost;dbname=app',
        'user',
        'pass',
        [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
    );
});

// Auto-resolve with dependencies
$userService = $container->make(UserService::class);
// Container automatically injects:
// - UserRepositoryInterface (resolved to MySQLUserRepository)
// - MailerInterface (resolved to SmtpMailer)
```

## Container-Aware Classes (Avoid)

```php
<?php
// BAD: Service locator anti-pattern
class BadService {
    public function __construct(private Container $container) {}
    
    public function doSomething(): void {
        $repo = $this->container->make(UserRepository::class);  // Hidden dependency!
    }
}

// GOOD: Explicit dependencies
class GoodService {
    public function __construct(private UserRepository $repo) {}  // Visible!
}
```

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*