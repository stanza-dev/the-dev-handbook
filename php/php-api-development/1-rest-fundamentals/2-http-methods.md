---
source_course: "php-api-development"
source_lesson: "php-api-development-http-methods"
---

# HTTP Methods and Status Codes

## Introduction
HTTP methods and status codes are the vocabulary of every REST API. Methods tell the server what operation to perform, and status codes tell the client what happened. Mastering both is a prerequisite for building APIs that other developers can understand without guessing.

## Key Concepts
- **HTTP Method (Verb)**: A keyword in the request line (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) that indicates the desired action on the resource.
- **Idempotency**: An operation is idempotent if calling it once or many times produces the same server state. `GET`, `PUT`, and `DELETE` are idempotent; `POST` is not. `PATCH` is not guaranteed to be idempotent.
- **Safe Method**: A method is safe if it does not modify server state. Only `GET` and `HEAD` are safe.
- **Status Code**: A three-digit number in the response indicating success, client error, or server error.

## Real World Context
Payment gateways like Stripe rely on idempotency to prevent double charges. If a network timeout occurs after a `POST /charges` request, the client can safely retry with an idempotency key because the server knows to return the original result instead of creating a second charge. Understanding these semantics is critical for building reliable systems.

## Deep Dive

### The Five Core Methods
Every REST API revolves around five HTTP methods. Here is how they map to typical CRUD operations:

```php
<?php
declare(strict_types=1);

$method = $_SERVER['REQUEST_METHOD'];

match ($method) {
    'GET'    => handleGet(),    // Read — safe, idempotent
    'POST'   => handlePost(),   // Create — NOT idempotent
    'PUT'    => handlePut(),    // Replace — idempotent
    'PATCH'  => handlePatch(),  // Partial update — not necessarily idempotent
    'DELETE' => handleDelete(), // Remove — idempotent
    default  => methodNotAllowed(),
};

function methodNotAllowed(): never {
    http_response_code(405);
    header('Allow: GET, POST, PUT, PATCH, DELETE');
    echo json_encode(['error' => 'Method not allowed']);
    exit;
}
```

The `match` expression maps each method to a handler. Notice the 405 response when an unsupported method arrives — the `Allow` header tells the client which methods are valid.

### PUT vs PATCH
A common source of confusion is the difference between `PUT` and `PATCH`:

```php
<?php
// PUT replaces the entire resource.
// If you omit a field, the server sets it to null.
// PUT /users/42
$putBody = [
    'name'  => 'Alice Johnson',
    'email' => 'alice@example.com',
    'bio'   => 'Software engineer',  // must include ALL fields
];

// PATCH updates only the supplied fields.
// Omitted fields remain unchanged.
// PATCH /users/42
$patchBody = [
    'bio' => 'Senior software engineer',  // only the changed field
];
```

`PUT` is a full replacement; `PATCH` is a partial update. Use `PATCH` when clients should be able to update individual fields without resending the entire resource.

### Status Codes You Must Know
Status codes are grouped by the first digit:

| Range | Category | Common Codes |
|---|---|---|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx | Server Error | 500 Internal Server Error |

Here is a helper function that returns the right status code for each scenario:

```php
<?php
declare(strict_types=1);

function respond(mixed $data, int $status = 200): never {
    http_response_code($status);
    header('Content-Type: application/json; charset=utf-8');

    if ($status === 204) {
        // 204 No Content must not have a body
        exit;
    }

    echo json_encode($data, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE);
    exit;
}

// 200 OK — successful read
respond(['data' => $user], 200);

// 201 Created — resource was created
respond(['data' => $newUser], 201);

// 204 No Content — successful delete, nothing to return
respond(null, 204);

// 400 Bad Request — malformed input
respond(['error' => 'Invalid JSON in request body'], 400);

// 401 Unauthorized — missing or invalid credentials
respond(['error' => 'Authentication required'], 401);

// 403 Forbidden — authenticated but not allowed
respond(['error' => 'You do not have permission'], 403);

// 404 Not Found — resource does not exist
respond(['error' => 'User not found'], 404);

// 500 Internal Server Error — unexpected failure
respond(['error' => 'Something went wrong'], 500);
```

Each status code communicates precise meaning. Clients use these codes to decide what to do next (retry, show an error, redirect, etc.).

## Common Pitfalls
1. **Returning 200 for everything** — Some APIs return `200 OK` with an error message in the body. This forces clients to parse the body to detect failures. Always use the correct status code so clients can branch on the HTTP status alone.
2. **Using POST for updates** — POST is for creation. Use PUT or PATCH for updates. Misusing POST breaks idempotency expectations and confuses API consumers.

## Best Practices
1. **Return 201 with a Location header after creation** — When `POST` creates a resource, respond with `201 Created` and a `Location` header pointing to the new resource's URL. This follows the HTTP specification and lets clients immediately navigate to the created resource.
2. **Use 204 for successful deletes** — After deleting a resource, return `204 No Content` with an empty body. There is no resource left to return, so an empty response is the correct semantic choice.

## Summary
- HTTP methods (GET, POST, PUT, PATCH, DELETE) define the action; URLs define the resource.
- GET is safe and idempotent; POST is neither. PUT and DELETE are idempotent but not safe. PATCH is not guaranteed to be idempotent.
- Use specific status codes: 200 for reads, 201 for creation, 204 for deletion, 4xx for client errors, 5xx for server errors.
- PUT replaces the entire resource; PATCH updates only the supplied fields.
- Always include meaningful status codes so clients can react without parsing the response body.

## Code Examples

**A /products endpoint demonstrating GET (200), POST (201 with Location header), DELETE (204), and error handling (400, 404)**

```php
<?php
declare(strict_types=1);

// Demonstrates correct HTTP method handling and status codes
// for a /products resource.

$method = $_SERVER['REQUEST_METHOD'];
$uri    = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

header('Content-Type: application/json; charset=utf-8');

// Simulated product store
$products = [
    1 => ['id' => 1, 'name' => 'Keyboard', 'price' => 79.99],
    2 => ['id' => 2, 'name' => 'Monitor',  'price' => 349.00],
];

if ($uri === '/products' && $method === 'GET') {
    // 200 OK — return the collection
    http_response_code(200);
    echo json_encode(['data' => array_values($products)]);

} elseif ($uri === '/products' && $method === 'POST') {
    // Read and validate the request body
    $body = json_decode(file_get_contents('php://input'), true);

    if (!isset($body['name'], $body['price'])) {
        http_response_code(400); // 400 Bad Request
        echo json_encode(['error' => 'name and price are required']);
        exit;
    }

    $newId   = max(array_keys($products)) + 1;
    $product = ['id' => $newId, 'name' => $body['name'], 'price' => (float) $body['price']];

    // 201 Created — include Location header
    http_response_code(201);
    header("Location: /products/{$newId}");
    echo json_encode(['data' => $product]);

} elseif (preg_match('#^/products/(\d+)$#', $uri, $m) && $method === 'DELETE') {
    $id = (int) $m[1];
    // 204 No Content — resource deleted, empty body
    http_response_code(204);
    // No echo — 204 must not have a body

} else {
    http_response_code(404);
    echo json_encode(['error' => 'Not found']);
}
?>
```


## Resources

- [http_response_code — PHP Manual](https://www.php.net/manual/en/function.http-response-code.php) — Official PHP reference for getting and setting HTTP status codes
- [HTTP request methods — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) — Mozilla reference covering every HTTP method, its semantics, and idempotency

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*