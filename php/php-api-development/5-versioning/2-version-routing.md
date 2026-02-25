---
source_course: "php-api-development"
source_lesson: "php-api-development-version-routing"
---

# Implementing Version Routing

## Introduction
Choosing a versioning strategy is only half the battle. You also need a clean routing architecture that dispatches requests to the correct version without duplicating your entire codebase. This lesson shows you how to build a version-aware router that keeps shared logic centralized.

## Key Concepts
- **Version Router**: A router component that extracts the version from the request and dispatches to the appropriate controller.
- **Version Negotiation**: The process of determining which API version a client is requesting.
- **Backward Compatibility**: Ensuring older API versions continue to work correctly when new versions are deployed.
- **Controller Inheritance**: Using a base controller to share common logic across versioned controllers.

## Real World Context
In a growing API, you might have 50 endpoints across 3 versions. Without a proper routing strategy, you end up copy-pasting controllers and maintaining triple the code. A well-designed version router lets you share 80% of the logic and only override what changes between versions.

## Deep Dive

### Building a Version-Aware Router

A version router extracts the version prefix from the URI and forwards the request to the appropriate handler. This approach centralizes version logic in one place.

Here is a complete version-aware router class:

```php
<?php
class VersionedRouter
{
    /** @var array<int, array<string, callable>> */
    private array $routes = [];
    private int $defaultVersion = 1;

    public function register(
        int $version,
        string $method,
        string $path,
        callable $handler
    ): void {
        $key = strtoupper($method) . ':' . $path;
        $this->routes[$version][$key] = $handler;
    }

    public function dispatch(string $method, string $uri): mixed
    {
        // Extract version from URI: /api/v2/products -> version=2
        if (!preg_match('#^/api/v(\d+)(/.*)$#', $uri, $matches)) {
            throw new RuntimeException('URI must start with /api/v{n}/');
        }

        $version = (int) $matches[1];
        $path = $matches[2];
        $key = strtoupper($method) . ':' . $path;

        // Look up handler for this version, fall back to previous versions
        for ($v = $version; $v >= 1; $v--) {
            if (isset($this->routes[$v][$key])) {
                return ($this->routes[$v][$key])();
            }
        }

        throw new RuntimeException("No route found: $method $uri");
    }
}
```

The `dispatch` method includes a fallback mechanism: if version 3 does not define a route, it falls back to version 2, then version 1. This means you only register overrides for endpoints that actually change.

### Registering Versioned Routes

With the router in place, you register endpoints per version. Unchanged endpoints inherit from earlier versions automatically.

Here is how you register routes across two versions:

```php
<?php
$router = new VersionedRouter();

// V1 endpoints
$router->register(1, 'GET', '/products', function () {
    $controller = new ProductControllerV1();
    return $controller->index();
});

$router->register(1, 'GET', '/products/{id}', function () {
    $controller = new ProductControllerV1();
    return $controller->show();
});

// V2 overrides only the index endpoint
$router->register(2, 'GET', '/products', function () {
    $controller = new ProductControllerV2();
    return $controller->index();
});

// GET /api/v2/products       -> ProductControllerV2::index()
// GET /api/v2/products/{id}  -> ProductControllerV1::show() (inherited)
```

Version 2 only overrides the `index` endpoint. The `show` endpoint falls back to V1 automatically, reducing code duplication.

### Shared Logic with Base Controllers

Avoid duplicating business logic by using a base controller that versioned controllers extend:

```php
<?php
abstract class BaseProductController extends Controller
{
    // Shared validation and business logic
    protected function validateProduct(array $data): array
    {
        $errors = [];
        if (empty($data['name'])) {
            $errors[] = 'Product name is required';
        }
        if (!isset($data['price']) || $data['price'] < 0) {
            $errors[] = 'Price must be a non-negative number';
        }
        return $errors;
    }

    // Each version provides its own response format
    abstract protected function formatProduct(Product $product): array;
}

class ProductControllerV1 extends BaseProductController
{
    protected function formatProduct(Product $product): array
    {
        return [
            'id' => $product->id,
            'name' => $product->name,
            'price' => $product->price,
        ];
    }
}

class ProductControllerV2 extends BaseProductController
{
    protected function formatProduct(Product $product): array
    {
        return [
            'id' => $product->id,
            'name' => $product->name,
            'price' => $product->price,
            'currency' => $product->currency,
            'created_at' => $product->createdAt->format('c'),
        ];
    }
}
```

The `BaseProductController` holds shared validation logic. Each version only overrides the `formatProduct` method to shape the response differently. This keeps your code DRY while allowing response formats to diverge between versions.

### Deprecation Headers

When sunsetting an old version, include deprecation headers so clients know to migrate:

```php
<?php
function addDeprecationHeaders(int $version): void
{
    $deprecatedVersions = [
        1 => '2026-06-01',
        2 => '2027-01-01',
    ];

    if (isset($deprecatedVersions[$version])) {
        header('Deprecation: true');
        header('Sunset: ' . $deprecatedVersions[$version]);
        header('Link: </api/v3/docs>; rel="successor-version"');
    }
}
```

The `Sunset` header tells clients exactly when the version will be removed, giving them a clear migration deadline.

## Common Pitfalls
1. **Duplicating entire controllers per version** — Copy-pasting all endpoints into every version creates a maintenance nightmare. Use inheritance or composition to share unchanged logic and only override what differs.
2. **Not testing older versions after changes** — Shared code changes can silently break older versions. Always run your test suite against all supported versions after modifications.

## Best Practices
1. **Use fallback routing** — Only register routes for versions where behavior changes. Let unchanged endpoints inherit from previous versions to minimize duplication.
2. **Communicate deprecation proactively** — Add `Sunset` and `Deprecation` headers to old versions and document migration timelines in your API changelog.

## Summary
- A version-aware router extracts the version from the request and dispatches to the correct handler.
- Fallback routing lets newer versions inherit unchanged endpoints from older versions.
- Base controllers with abstract methods keep shared logic centralized while allowing response formats to vary.
- Deprecation headers (`Sunset`, `Deprecation`) warn clients about upcoming version removals.

## Code Examples

**Version-aware router with automatic fallback — V2 only overrides the index endpoint while inheriting everything else from V1**

```php
<?php
// Version-aware router with fallback to previous versions
$router = new VersionedRouter();

// V1: all endpoints defined
$router->register(1, 'GET', '/products', [ProductControllerV1::class, 'index']);
$router->register(1, 'POST', '/products', [ProductControllerV1::class, 'store']);
$router->register(1, 'GET', '/products/{id}', [ProductControllerV1::class, 'show']);

// V2: only override what changed
$router->register(2, 'GET', '/products', [ProductControllerV2::class, 'index']);
// POST and show inherit from V1 automatically

// Dispatch: GET /api/v2/products/{id} -> falls back to V1::show()
$response = $router->dispatch('GET', '/api/v2/products/42');
```


## Resources

- [PHP preg_match — Regular Expressions](https://www.php.net/manual/en/function.preg-match.php) — Official PHP documentation for regex used in URI version extraction

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*