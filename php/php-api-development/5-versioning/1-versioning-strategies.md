---
source_course: "php-api-development"
source_lesson: "php-api-development-versioning-strategies"
---

# API Versioning Strategies

## Introduction
As your API evolves, you need a strategy to introduce breaking changes without disrupting existing consumers. API versioning gives clients a stable contract they can rely on while you iterate on new features and improvements behind the scenes.

## Key Concepts
- **API Versioning**: A technique for managing breaking changes by allowing multiple API versions to coexist simultaneously.
- **URI Versioning**: Embedding the version number directly in the URL path, such as `/v1/users` or `/v2/users`.
- **Header Versioning**: Specifying the desired API version through HTTP headers like `Accept` or a custom `X-API-Version` header.
- **Query Parameter Versioning**: Passing the version as a query string parameter, e.g. `?version=2`.
- **Breaking Change**: Any modification that would cause existing API consumers to fail, such as removing a field or changing a response structure.

## Real World Context
Every mature API faces the versioning question. Stripe, GitHub, and Twilio all use versioning to ship improvements without breaking millions of integrations. Without a versioning strategy, a single field rename could take down every client consuming your API overnight.

## Deep Dive

### URI Versioning (Path-Based)

URI versioning is the most popular approach. The version number becomes part of the URL path, making it immediately visible and easy to test in a browser.

Here is how you set up versioned route groups in PHP:

```php
<?php
// URI versioning: the version lives in the URL path
// /api/v1/products -> ProductControllerV1
// /api/v2/products -> ProductControllerV2

$router->group('/api/v1', function (Router $router) {
    $router->get('/products', [ProductControllerV1::class, 'index']);
    $router->get('/products/{id}', [ProductControllerV1::class, 'show']);
    $router->post('/products', [ProductControllerV1::class, 'store']);
});

$router->group('/api/v2', function (Router $router) {
    $router->get('/products', [ProductControllerV2::class, 'index']);
    $router->get('/products/{id}', [ProductControllerV2::class, 'show']);
    $router->post('/products', [ProductControllerV2::class, 'store']);
});
```

Each version maps to its own controller class. V1 and V2 can return different response shapes without interfering with each other. This is the approach used by GitHub (`api.github.com/v3/`) and most public APIs.

### Header Versioning

Header versioning keeps URLs clean by moving the version into an HTTP header. Clients set a custom header or use the `Accept` header with a vendor media type.

The following function extracts the version from incoming request headers:

```php
<?php
function resolveApiVersion(): int
{
    // Option 1: Custom header (X-API-Version: 2)
    $customHeader = $_SERVER['HTTP_X_API_VERSION'] ?? null;
    if ($customHeader !== null) {
        return (int) $customHeader;
    }

    // Option 2: Accept header (Accept: application/vnd.myapi.v2+json)
    $accept = $_SERVER['HTTP_ACCEPT'] ?? '';
    if (preg_match('/vnd\.myapi\.v(\d+)\+json/', $accept, $matches)) {
        return (int) $matches[1];
    }

    // Default to version 1 if no header is provided
    return 1;
}

$version = resolveApiVersion();
```

This approach is favored by APIs that want stable URLs. The downside is that version negotiation is invisible in the URL, making it harder to test with a browser or share links.

### Query Parameter Versioning

Query parameter versioning appends the version as a URL parameter. It is the simplest to implement but the least conventional for production APIs.

Here is a minimal implementation:

```php
<?php
// Query parameter: /api/products?version=2
$requestedVersion = (int) ($_GET['version'] ?? 1);

$controllerMap = [
    1 => ProductControllerV1::class,
    2 => ProductControllerV2::class,
];

$controllerClass = $controllerMap[$requestedVersion]
    ?? throw new InvalidArgumentException(
        "Unsupported API version: $requestedVersion"
    );
```

While easy to understand, query parameters are typically reserved for filtering and pagination, so mixing version numbers in can be confusing for API consumers.

### Comparing the Three Approaches

| Strategy | Visibility | Cacheability | URL Stability | Adoption |
|----------|-----------|-------------|---------------|----------|
| URI Path | High | Excellent | URLs change per version | Most common |
| Header | Low | Requires Vary header | URLs stay the same | Medium |
| Query Param | Medium | Poor (cache key varies) | URLs change slightly | Least common |

URI versioning is the industry standard for public APIs because it is explicit, cacheable, and easy to document.

## Common Pitfalls
1. **Versioning too aggressively** — Creating a new version for every minor change leads to version sprawl. Only version when you introduce breaking changes; additive changes (new fields, new endpoints) do not require a new version.
2. **Forgetting to deprecate old versions** — Supporting every version indefinitely creates maintenance burden. Communicate a deprecation timeline (e.g., 12 months) and return `Sunset` headers to warn consumers.

## Best Practices
1. **Start with URI versioning** — It is the most widely understood approach and works well with API gateways, documentation tools, and caching layers.
2. **Use semantic versioning for internal tracking** — Even if your public API uses simple v1/v2 numbering, track internal changes with semver to know when a bump is truly breaking.

## Summary
- API versioning lets you evolve your API without breaking existing consumers.
- URI versioning (`/v1/`, `/v2/`) is the most popular and cache-friendly approach.
- Header versioning keeps URLs clean but hides the version from casual inspection.
- Query parameter versioning is simple but unconventional for production APIs.
- Only create a new version for breaking changes; additive changes are backward-compatible.

## Code Examples

**URI versioning with route groups — V2 adds review data while V1 remains unchanged for existing consumers**

```php
<?php
// URI versioning with route groups
$router->group('/api/v1', function (Router $router) {
    $router->get('/products', [ProductControllerV1::class, 'index']);
    $router->get('/products/{id}', [ProductControllerV1::class, 'show']);
});

$router->group('/api/v2', function (Router $router) {
    $router->get('/products', [ProductControllerV2::class, 'index']);
    $router->get('/products/{id}', [ProductControllerV2::class, 'show']);
});

// V2 returns enriched data without breaking V1 clients
class ProductControllerV2 extends Controller
{
    public function show(int $id): never
    {
        $product = $this->products->findWithReviews($id);

        $this->json([
            'data' => [
                'id' => $product->id,
                'name' => $product->name,
                'price' => $product->price,
                'reviews' => $product->reviews, // New in V2
                'rating' => $product->averageRating, // New in V2
            ],
        ]);
    }
}
```


## Resources

- [PHP preg_match — Pattern Matching](https://www.php.net/manual/en/function.preg-match.php) — Official PHP documentation for regex matching used in header versioning

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*