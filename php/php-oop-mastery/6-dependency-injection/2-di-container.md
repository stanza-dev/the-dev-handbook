---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-di-container"
---

# Building a Simple DI Container

## Introduction
A DI Container manages object creation and dependency resolution automatically. Instead of manually wiring dependencies, you register bindings and the container resolves them recursively using Reflection.

## Key Concepts
- **Binding**: Mapping an abstract type (interface) to a concrete implementation.
- **Resolution**: The container creates objects by inspecting constructor parameters.
- **Singleton**: A binding that returns the same instance every time.
- **Auto-Wiring**: Automatically resolving dependencies based on type hints.

## Real World Context
In a web application with dozens of services, manually wiring every dependency becomes unwieldy. A DI container reads type hints from constructors and automatically creates and injects the right objects. Frameworks like Laravel and Symfony include powerful DI containers.

## Deep Dive

### Basic Container

```php
<?php
class Container {
    private array $bindings = [];
    private array $instances = [];

    public function bind(string $abstract, string|callable $concrete): void {
        $this->bindings[$abstract] = $concrete;
    }

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

    public function make(string $abstract): object {
        if (isset($this->bindings[$abstract])) {
            $concrete = $this->bindings[$abstract];
            if (is_callable($concrete)) {
                return $concrete($this);
            }
            return $this->make($concrete);
        }
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
                    throw new Exception("Cannot resolve: {$param->getName()}");
                }
            } else {
                $dependencies[] = $this->make($type->getName());
            }
        }

        return $reflection->newInstanceArgs($dependencies);
    }
}
```

The container uses Reflection to inspect constructors and recursively resolve dependencies.

### Using the Container

```php
<?php
$container = new Container();

$container->bind(UserRepositoryInterface::class, MySQLUserRepository::class);
$container->bind(MailerInterface::class, SmtpMailer::class);

$container->singleton(PDO::class, function() {
    return new PDO(
        'mysql:host=localhost;dbname=app',
        'user',
        'pass',
        [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
    );
});

// Auto-resolve with all dependencies injected
$userService = $container->make(UserService::class);
```

The container automatically injects the right implementations based on registered bindings.

### Service Locator Anti-Pattern

```php
<?php
// BAD: Service locator - hides dependencies
class BadService {
    public function __construct(private Container $container) {}

    public function doSomething(): void {
        $repo = $this->container->make(UserRepository::class);  // Hidden!
    }
}

// GOOD: Explicit dependencies
class GoodService {
    public function __construct(private UserRepository $repo) {}  // Visible!
}
```

Never inject the container itself. This hides real dependencies and makes code harder to understand and test.

## Common Pitfalls
1. **Circular dependencies** — A depends on B, B depends on A. The container will overflow. Refactor to break the cycle.
2. **Service Locator pattern** — Injecting the container itself hides real dependencies. Always inject specific interfaces.

## Best Practices
1. **Use singleton for shared resources** — Database connections, loggers, and caches should be singletons.
2. **Register interfaces, not classes** — Bind interfaces to implementations for easy swapping.

## Summary
- DI containers automate dependency resolution using Reflection.
- Bindings map interfaces to concrete implementations.
- Singletons ensure shared resources are created only once.
- Never inject the container itself; always inject specific dependencies.

## Code Examples

**Auto-wiring dependencies with a simple DI container**

```php
<?php
declare(strict_types=1);

interface Logger {
    public function log(string $message): void;
}

class FileLogger implements Logger {
    public function log(string $message): void {
        echo "[File] $message
";
    }
}

class UserService {
    public function __construct(
        private Logger $logger
    ) {}

    public function createUser(string $name): void {
        $this->logger->log("Creating user: $name");
    }
}

// Simple container usage
$container = new Container();
$container->bind(Logger::class, FileLogger::class);

// Auto-resolves: UserService needs Logger, gets FileLogger
$userService = $container->make(UserService::class);
$userService->createUser("John");
?>
```


## Resources

- [PHP-DI](https://php-di.org/) — PHP-DI dependency injection container

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*