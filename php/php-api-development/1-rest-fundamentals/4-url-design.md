---
source_course: "php-api-development"
source_lesson: "php-api-development-url-design"
---

# URL Design and Resource Naming

## Introduction
Well-designed URLs make an API intuitive and self-documenting. A developer should be able to guess the URL for a resource without reading documentation. This lesson covers RESTful URL conventions, nested resources, query parameters, and how PHP 8.5's new Uri class simplifies URL manipulation.

## Key Concepts
- **Collection URL**: A URL that represents a group of resources, using a plural noun (e.g., `/products`).
- **Resource URL**: A URL that identifies a single resource by its ID (e.g., `/products/42`).
- **Nested resource**: A resource that belongs to a parent, represented by nesting in the URL (e.g., `/users/7/orders`).
- **Query parameter**: A key-value pair after `?` in the URL used for filtering, sorting, or pagination (e.g., `?page=2&sort=price`).

## Real World Context
The GitHub API is a model of excellent URL design. To list a user's repositories you call `GET /users/octocat/repos`. To get a specific repo: `GET /repos/octocat/hello-world`. The URLs are predictable and consistent — thousands of developers build integrations without needing to memorise every endpoint.

## Deep Dive

### Plural Nouns, Not Verbs
The most fundamental rule: use plural nouns for collection endpoints and let HTTP methods express the action.

```php
<?php
// CORRECT: nouns in the URL, verbs via HTTP methods
// GET    /products       → list products
// POST   /products       → create a product
// GET    /products/42    → read product 42
// PATCH  /products/42    → update product 42
// DELETE /products/42    → delete product 42

// WRONG: verbs in the URL
// GET  /getProducts
// POST /createProduct
// POST /deleteProduct/42
```

When the URL contains only nouns, the HTTP method unambiguously defines the intent.

### Nesting Resources
When a resource logically belongs to another, nest it one level deep:

```php
<?php
// Good: one level of nesting
// GET /users/7/orders       → orders belonging to user 7
// GET /users/7/orders/42    → order 42 of user 7

// Avoid: deep nesting becomes unreadable
// GET /users/7/orders/42/items/3/reviews/12
// Better: flatten and use query filters
// GET /reviews?order_item_id=3
```

One level of nesting makes the parent-child relationship clear. Beyond that, flatten the URL and use query parameters to filter.

### Query Parameters for Filtering, Sorting, Pagination
Use query parameters to refine collection responses:

```php
<?php
declare(strict_types=1);

// GET /products?category=electronics&sort=price&order=asc&page=2&limit=20

$category = $_GET['category'] ?? null;
$sort     = $_GET['sort']     ?? 'id';
$order    = $_GET['order']    ?? 'asc';
$page     = max(1, (int) ($_GET['page']  ?? 1));
$limit    = min(100, max(1, (int) ($_GET['limit'] ?? 20)));

// Build SQL with safe defaults
$allowedSorts  = ['id', 'name', 'price', 'created_at'];
$allowedOrders = ['asc', 'desc'];

$sort  = in_array($sort, $allowedSorts, true)   ? $sort  : 'id';
$order = in_array($order, $allowedOrders, true) ? $order : 'asc';
$offset = ($page - 1) * $limit;

// The query parameters keep the URL clean and the logic flexible
```

Always validate and whitelist sort columns to prevent SQL injection through query parameters.

### PHP 8.5 Uri Class
PHP 8.5 introduces a built-in `Uri` class in the `Uri` extension that makes parsing and building URLs type-safe:

```php
<?php
declare(strict_types=1);

// PHP 8.5 Uri class for safe URL manipulation
use Uri\WhatWg\Url;

// Parse a full API URL
$uri = new Url('https://api.example.com/v1/products?category=books&page=2');

echo $uri->getScheme();   // "https"
echo $uri->getHost();     // "api.example.com"
echo $uri->getPath();     // "/v1/products"
echo $uri->getQuery();    // "category=books&page=2"

// Build a new URL by modifying components
$nextPage = $uri->withQuery('category=books&page=3');
echo (string) $nextPage;
// Output: https://api.example.com/v1/products?category=books&page=3
```

The `Uri` extension replaces ad-hoc `parse_url()` calls with an object-oriented API that handles edge cases like encoding and normalisation automatically.

### Consistent URL Patterns in a Router
Here is how URL design translates into router definitions:

```php
<?php
declare(strict_types=1);

// A well-designed set of routes following REST conventions
$router->get('/products',              [ProductController::class, 'index']);
$router->post('/products',             [ProductController::class, 'store']);
$router->get('/products/{id}',         [ProductController::class, 'show']);
$router->patch('/products/{id}',       [ProductController::class, 'update']);
$router->delete('/products/{id}',      [ProductController::class, 'destroy']);

// Nested: a product's reviews
$router->get('/products/{id}/reviews', [ReviewController::class, 'index']);
$router->post('/products/{id}/reviews',[ReviewController::class, 'store']);

// Filtering via query parameters, not extra path segments
// GET /products?category=electronics&sort=price
```

Every endpoint follows the same pattern: plural resource name, optional `{id}`, and at most one level of nesting.

## Common Pitfalls
1. **Inconsistent pluralisation** — Mixing `/product/42` and `/users` in the same API is confusing. Pick plural nouns and stick with them across every resource.
2. **Encoding user input in URLs unsafely** — When building URLs from dynamic values, always use `rawurlencode()` to prevent injection. PHP 8.5's `Uri` class handles this automatically.

## Best Practices
1. **Version your API in the URL** — Prefix all routes with `/v1/` so you can introduce breaking changes in `/v2/` without disrupting existing clients.
2. **Limit nesting to one level** — `/users/{id}/orders` is fine. If you need deeper access, flatten the URL and use query parameters: `/order-items?order_id=42` instead of `/users/7/orders/42/items`.

## Summary
- Use plural nouns for collection URLs (`/products`) and append an ID for individual resources (`/products/42`).
- Nest resources at most one level deep to express parent-child relationships.
- Use query parameters for filtering, sorting, and pagination.
- Validate and whitelist all query parameter values before using them in database queries.
- PHP 8.5's `Uri` class provides a type-safe, object-oriented way to parse and build URLs.

## Code Examples

**Paginated collection endpoint with query parameter validation, metadata, and HATEOAS-style pagination links**

```php
<?php
declare(strict_types=1);

// Demonstrates RESTful URL design with pagination metadata

// GET /products?category=electronics&page=2&limit=10
$category = $_GET['category'] ?? null;
$page     = max(1, (int) ($_GET['page']  ?? 1));
$limit    = min(100, max(1, (int) ($_GET['limit'] ?? 10)));
$offset   = ($page - 1) * $limit;

// Simulated database result
$totalProducts = 47;
$products = [
    ['id' => 11, 'name' => 'USB-C Cable',  'price' => 12.99],
    ['id' => 12, 'name' => 'Webcam HD',    'price' => 59.99],
];

// Build pagination links (HATEOAS-style)
$baseUrl   = '/products';
$query     = $category ? "category={$category}&" : '';
$lastPage  = (int) ceil($totalProducts / $limit);

$links = [
    'self'  => "{$baseUrl}?{$query}page={$page}&limit={$limit}",
    'first' => "{$baseUrl}?{$query}page=1&limit={$limit}",
    'last'  => "{$baseUrl}?{$query}page={$lastPage}&limit={$limit}",
];

if ($page > 1) {
    $links['prev'] = "{$baseUrl}?{$query}page=" . ($page - 1) . "&limit={$limit}";
}
if ($page < $lastPage) {
    $links['next'] = "{$baseUrl}?{$query}page=" . ($page + 1) . "&limit={$limit}";
}

header('Content-Type: application/json; charset=utf-8');
echo json_encode([
    'data'  => $products,
    'meta'  => [
        'current_page' => $page,
        'per_page'     => $limit,
        'total'        => $totalProducts,
        'last_page'    => $lastPage,
    ],
    '_links' => $links,
], JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT);
?>
```


## Resources

- [parse_url — PHP Manual](https://www.php.net/manual/en/function.parse-url.php) — PHP reference for parsing URL components
- [rawurlencode — PHP Manual](https://www.php.net/manual/en/function.rawurlencode.php) — PHP reference for percent-encoding URL components according to RFC 3986

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*