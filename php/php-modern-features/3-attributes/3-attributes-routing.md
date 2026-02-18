---
source_course: "php-modern-features"
source_lesson: "php-modern-features-attributes-routing"
---

# Building an Attribute-Based Router

Attributes enable declarative routing similar to frameworks like Symfony or Laravel. Let's build a complete routing system.

## Route Attribute Definition

```php
<?php
#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET',
        public ?string $name = null,
        public array $middleware = []
    ) {}
}

#[Attribute(Attribute::TARGET_CLASS)]
class Controller
{
    public function __construct(
        public string $prefix = ''
    ) {}
}

#[Attribute(Attribute::TARGET_METHOD)]
class Middleware
{
    public function __construct(
        public string $name
    ) {}
}
```

## Controller Definition

```php
<?php
#[Controller('/api/users')]
class UserController
{
    #[Route('/', 'GET', name: 'users.index')]
    #[Middleware('auth')]
    public function index(): array
    {
        return ['users' => []];
    }
    
    #[Route('/{id}', 'GET', name: 'users.show')]
    #[Middleware('auth')]
    public function show(int $id): array
    {
        return ['user' => ['id' => $id]];
    }
    
    #[Route('/', 'POST', name: 'users.create')]
    #[Middleware('auth')]
    #[Middleware('admin')]
    public function create(): array
    {
        return ['created' => true];
    }
}
```

## Route Collection

```php
<?php
class RouteDefinition
{
    public function __construct(
        public string $path,
        public string $method,
        public string $controller,
        public string $action,
        public ?string $name,
        public array $middleware
    ) {}
    
    public function matches(string $method, string $path): ?array
    {
        if ($this->method !== $method) {
            return null;
        }
        
        // Convert {param} to regex
        $pattern = preg_replace('/\{(\w+)\}/', '(?P<$1>[^/]+)', $this->path);
        $pattern = '#^' . $pattern . '$#';
        
        if (preg_match($pattern, $path, $matches)) {
            return array_filter($matches, 'is_string', ARRAY_FILTER_USE_KEY);
        }
        
        return null;
    }
}
```

## Router Implementation

```php
<?php
class AttributeRouter
{
    /** @var RouteDefinition[] */
    private array $routes = [];
    
    public function registerController(string $className): void
    {
        $reflection = new ReflectionClass($className);
        
        // Get controller prefix
        $prefix = '';
        $controllerAttrs = $reflection->getAttributes(Controller::class);
        if (!empty($controllerAttrs)) {
            $prefix = $controllerAttrs[0]->newInstance()->prefix;
        }
        
        // Register each route method
        foreach ($reflection->getMethods(ReflectionMethod::IS_PUBLIC) as $method) {
            $routeAttrs = $method->getAttributes(Route::class);
            
            foreach ($routeAttrs as $attr) {
                $route = $attr->newInstance();
                
                // Collect middleware
                $middleware = $route->middleware;
                foreach ($method->getAttributes(Middleware::class) as $mwAttr) {
                    $middleware[] = $mwAttr->newInstance()->name;
                }
                
                $this->routes[] = new RouteDefinition(
                    path: $prefix . $route->path,
                    method: $route->method,
                    controller: $className,
                    action: $method->getName(),
                    name: $route->name,
                    middleware: $middleware
                );
            }
        }
    }
    
    public function dispatch(string $method, string $uri): mixed
    {
        foreach ($this->routes as $route) {
            $params = $route->matches($method, $uri);
            
            if ($params !== null) {
                // Run middleware
                foreach ($route->middleware as $mw) {
                    $this->runMiddleware($mw);
                }
                
                // Call controller action
                $controller = new ($route->controller)();
                return $controller->{$route->action}(...$params);
            }
        }
        
        throw new NotFoundException('Route not found');
    }
}

// Usage
$router = new AttributeRouter();
$router->registerController(UserController::class);

$result = $router->dispatch('GET', '/api/users/123');
// Calls UserController::show(123)
```

## Resources

- [Attributes](https://www.php.net/manual/en/language.attributes.php) — PHP attributes documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*