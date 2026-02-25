---
source_course: "php-modern-features"
source_lesson: "php-modern-features-attributes-routing"
---

# Building an Attribute-Based Router

## Introduction
Attributes enable declarative routing similar to Symfony, Laravel, and other modern frameworks. Instead of defining routes in a separate configuration file, you annotate controller methods directly with `#[Route]` attributes. This lesson walks through building a complete routing system.

## Key Concepts
- **Declarative Routing**: Defining routes as metadata on controller methods rather than in external configuration.
- **Route Registration**: The process of scanning controller classes for route attributes and building a lookup table.
- **Parameter Extraction**: Matching URL patterns like `/users/{id}` and extracting the `id` value.

## Real World Context
Symfony's routing component and Laravel's route attributes both use this pattern. By building one from scratch, you will understand the reflection-based scanning that happens behind the scenes in every modern PHP framework.

## Deep Dive

### Route Attribute Definition

First, define the attributes for routes, controllers, and middleware:

```php
<?php
#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class Route {
    public function __construct(
        public string $path,
        public string $method = 'GET',
        public ?string $name = null,
        public array $middleware = []
    ) {}
}

#[Attribute(Attribute::TARGET_CLASS)]
class Controller {
    public function __construct(public string $prefix = '') {}
}

#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class Middleware {
    public function __construct(public string $name) {}
}
```

The `Route` attribute is repeatable (a method can handle multiple routes) and method-targeted. The `Controller` attribute provides a URL prefix for all routes in the class.

### Controller Definition

Use the attributes on a controller:

```php
<?php
#[Controller('/api/users')]
class UserController {
    #[Route('/', 'GET', name: 'users.index')]
    #[Middleware('auth')]
    public function index(): array {
        return ['users' => []];
    }
    
    #[Route('/{id}', 'GET', name: 'users.show')]
    #[Middleware('auth')]
    public function show(int $id): array {
        return ['user' => ['id' => $id]];
    }
    
    #[Route('/', 'POST', name: 'users.create')]
    #[Middleware('auth')]
    #[Middleware('admin')]
    public function create(): array {
        return ['created' => true];
    }
}
```

Each method declares its route path, HTTP method, and required middleware. The `Controller` attribute provides the `/api/users` prefix.

### Router Implementation

The router scans controllers and builds a route table:

```php
<?php
class AttributeRouter {
    private array $routes = [];
    
    public function registerController(string $className): void {
        $reflection = new ReflectionClass($className);
        
        $prefix = '';
        $controllerAttrs = $reflection->getAttributes(Controller::class);
        if (!empty($controllerAttrs)) {
            $prefix = $controllerAttrs[0]->newInstance()->prefix;
        }
        
        foreach ($reflection->getMethods(ReflectionMethod::IS_PUBLIC) as $method) {
            $routeAttrs = $method->getAttributes(Route::class);
            
            foreach ($routeAttrs as $attr) {
                $route = $attr->newInstance();
                
                $middleware = $route->middleware;
                foreach ($method->getAttributes(Middleware::class) as $mwAttr) {
                    $middleware[] = $mwAttr->newInstance()->name;
                }
                
                $this->routes[] = [
                    'path' => $prefix . $route->path,
                    'method' => $route->method,
                    'controller' => $className,
                    'action' => $method->getName(),
                    'middleware' => $middleware,
                ];
            }
        }
    }
}

$router = new AttributeRouter();
$router->registerController(UserController::class);
```

The router reads the controller prefix, then scans each public method for `Route` attributes. It combines the prefix with the route path and collects middleware from both the route and separate `Middleware` attributes.

## Common Pitfalls
1. **Scanning too many classes** — Reflection on every class is expensive. Use file-system conventions or a class map to limit scanning to controller directories only.
2. **Route ordering matters** — A wildcard route like `/{slug}` can match before `/create`. Register specific routes before wildcards, or sort routes by specificity.

## Best Practices
1. **Cache the route table** — Scan controllers once during deployment or first request, serialize the route table, and load from cache on subsequent requests.
2. **Separate route definition from dispatch** — Build the route table in a boot phase. The dispatch phase should be a simple lookup, not involve reflection.

## Summary
- Attribute-based routing places route definitions on controller methods.
- A router scans controllers with reflection, reading `#[Route]` and `#[Middleware]` attributes.
- Controller-level `#[Controller('/prefix')]` provides URL prefixes.
- Cache the scanned route table for production performance.
- This pattern is used by Symfony, Laravel, and most modern PHP frameworks.

## Code Examples

**Minimal route registration function that scans a controller class for Route attributes using reflection**

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class Route {
    public function __construct(
        public string $path,
        public string $method = 'GET',
        public ?string $name = null
    ) {}
}

function registerRoutes(string $controllerClass): array {
    $routes = [];
    $reflection = new ReflectionClass($controllerClass);
    
    foreach ($reflection->getMethods(ReflectionMethod::IS_PUBLIC) as $method) {
        foreach ($method->getAttributes(Route::class) as $attr) {
            $route = $attr->newInstance();
            $routes[] = [
                'path' => $route->path,
                'method' => $route->method,
                'name' => $route->name,
                'handler' => [$controllerClass, $method->getName()],
            ];
        }
    }
    return $routes;
}
?>
```


## Resources

- [Attributes](https://www.php.net/manual/en/language.attributes.php) — PHP attributes documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*