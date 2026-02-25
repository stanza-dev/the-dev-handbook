---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-attributes-metaprogramming"
---

# Attributes & Metaprogramming

## Introduction
PHP 8 introduced native attributes as a structured way to add metadata to classes, methods, properties, and parameters. Combined with Reflection, attributes enable powerful metaprogramming patterns like route mapping, validation, serialization control, and dependency configuration.

## Key Concepts
- **Attributes**: Native PHP metadata declared with `#[...]` syntax.
- **Attribute Targets**: Attributes can target classes, methods, properties, parameters, or constants.
- **Repeatable Attributes**: An attribute can be applied multiple times to the same target.
- **Metaprogramming**: Writing code that operates on other code's structure.

## Real World Context
In a web framework, you want to map HTTP routes to controller methods. Instead of maintaining a separate routing file, you annotate controllers directly with `#[Route('/users', 'GET')]`. The framework reads these attributes at boot time and builds the route table automatically.

## Deep Dive

### Creating Custom Attributes

```php
<?php
#[Attribute(Attribute::TARGET_METHOD)]
class Route {
    public function __construct(
        public string $path,
        public string $method = 'GET'
    ) {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Column {
    public function __construct(
        public string $name,
        public string $type = 'varchar',
        public bool $nullable = false
    ) {}
}

#[Attribute(Attribute::TARGET_PARAMETER)]
class Inject {
    public function __construct(
        public string $service
    ) {}
}
```

The `#[Attribute]` attribute on the class declaration marks it as usable in attribute syntax.

### Route Mapping with Attributes

```php
<?php
#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class Route {
    public function __construct(
        public string $path,
        public string $method = 'GET'
    ) {}
}

class UserController {
    #[Route('/users', 'GET')]
    public function index(): array {
        return ['users' => []];
    }

    #[Route('/users', 'POST')]
    public function create(): array {
        return ['created' => true];
    }

    #[Route('/users/{id}', 'GET')]
    #[Route('/user/{id}', 'GET')]  // Alias route
    public function show(int $id): array {
        return ['user' => $id];
    }
}

class Router {
    private array $routes = [];

    public function registerController(string $class): void {
        $reflection = new ReflectionClass($class);

        foreach ($reflection->getMethods() as $method) {
            $attributes = $method->getAttributes(Route::class);

            foreach ($attributes as $attr) {
                $route = $attr->newInstance();
                $this->routes[] = [
                    'path' => $route->path,
                    'method' => $route->method,
                    'handler' => [$class, $method->getName()],
                ];
            }
        }
    }

    public function getRoutes(): array {
        return $this->routes;
    }
}

$router = new Router();
$router->registerController(UserController::class);
```

The router scans controller classes for Route attributes and builds the routing table automatically.

### Validation Attributes

```php
<?php
#[Attribute(Attribute::TARGET_PROPERTY | Attribute::IS_REPEATABLE)]
class NotBlank {
    public function __construct(public string $message = 'Must not be blank') {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Length {
    public function __construct(
        public int $min = 0,
        public int $max = PHP_INT_MAX,
        public string $message = 'Invalid length'
    ) {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Email {
    public function __construct(public string $message = 'Invalid email') {}
}

class RegistrationRequest {
    #[NotBlank]
    #[Length(min: 2, max: 50)]
    public string $name;

    #[NotBlank]
    #[Email]
    public string $email;

    #[NotBlank]
    #[Length(min: 8, message: 'Password must be at least 8 characters')]
    public string $password;
}
```

Validation rules are declared directly on the properties they validate.

### Combining Attributes with Enums

```php
<?php
enum Permission: string {
    case Read = 'read';
    case Write = 'write';
    case Admin = 'admin';
}

#[Attribute(Attribute::TARGET_METHOD)]
class RequiresPermission {
    public function __construct(
        public Permission $permission
    ) {}
}

class AdminController {
    #[RequiresPermission(Permission::Admin)]
    public function deleteUser(int $id): void {
        // ...
    }

    #[RequiresPermission(Permission::Read)]
    public function listUsers(): array {
        return [];
    }
}
```

Using enums in attributes provides type-safe, IDE-friendly metadata.

## Common Pitfalls
1. **Attribute logic in the attribute class** — Attributes should be pure data. Put processing logic in a separate handler.
2. **Overusing attributes** — Not everything needs to be an attribute. Use them for declarative metadata, not imperative logic.

## Best Practices
1. **Keep attributes as data** — Attributes declare what, not how. A separate processor reads and acts on them.
2. **Use IS_REPEATABLE wisely** — Allow multiple instances only when it makes semantic sense (like multiple routes).

## Summary
- Attributes provide native, type-safe metadata for PHP code.
- Combined with Reflection, they enable declarative patterns like routing and validation.
- Attributes should be pure data containers; processing logic belongs in handlers.
- Use enums in attributes for type-safe configuration values.

## Code Examples

**Declarative caching with attributes**

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_METHOD)]
class Cached {
    public function __construct(
        public int $ttl = 3600,
        public string $prefix = ''
    ) {}
}

class ProductService {
    #[Cached(ttl: 7200, prefix: 'products')]
    public function getPopular(): array {
        return ['Widget', 'Gadget', 'Doohickey'];
    }

    #[Cached(ttl: 300)]
    public function getLatest(): array {
        return ['New Widget'];
    }
}

// Read caching config from attributes
$ref = new ReflectionClass(ProductService::class);
foreach ($ref->getMethods() as $method) {
    $attrs = $method->getAttributes(Cached::class);
    foreach ($attrs as $attr) {
        $cached = $attr->newInstance();
        echo "{$method->getName()}: TTL={$cached->ttl}, prefix={$cached->prefix}\n";
    }
}
?>
```


## Resources

- [Attributes](https://www.php.net/manual/en/language.attributes.php) — PHP attributes documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*