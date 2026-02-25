---
source_course: "php-api-development"
source_lesson: "php-api-development-rest-principles"
---

# REST Architecture Principles

## Introduction
REST (Representational State Transfer) is the dominant architectural style behind modern web APIs. Understanding REST is essential for building APIs that are predictable, scalable, and easy for other developers to consume. In this lesson you will learn the core constraints that make an API truly RESTful and see how PHP 8.5 fits into the picture.

## Key Concepts
- **REST**: An architectural style that uses HTTP as its protocol, organizes data into resources, and relies on stateless communication between client and server.
- **Resource**: Any piece of data the API exposes, identified by a unique URL (e.g., `/users/42`).
- **Statelessness**: Every request from a client must carry all the information the server needs to fulfill it; the server never stores session context between requests.
- **HATEOAS**: Hypermedia As The Engine Of Application State — responses include links to related resources so clients can discover the API dynamically.

## Real World Context
Every time you use a mobile app that fetches your profile, posts, or notifications, a REST API is doing the heavy lifting behind the scenes. Companies like GitHub, Stripe, and Twilio expose REST APIs that thousands of developers integrate with daily. If your API follows REST constraints, consumers can predict its behavior without reading every page of documentation.

## Deep Dive
REST was introduced by Roy Fielding in his 2000 doctoral dissertation. It defines six constraints that, when applied together, produce APIs that scale well and remain maintainable over time.

### 1. Client-Server Separation
The client (browser, mobile app, CLI) handles the user interface. The server handles data storage and business logic. Neither side knows or cares about the internal details of the other.

Here is a minimal PHP script that acts purely as a server, returning data without any knowledge of how the client will display it:

```php
<?php
declare(strict_types=1);

// The server only cares about returning data.
// It has no opinion about how the client renders it.
header('Content-Type: application/json; charset=utf-8');
echo json_encode([
    'id'   => 42,
    'name' => 'Alice Johnson',
    'role' => 'engineer',
]);
```

The server sends a JSON payload. Whether the client is a React dashboard or a CLI tool is irrelevant to this code.

### 2. Statelessness
Each request is self-contained. The server never relies on information stored from a previous request.

Consider the difference between a stateful and a stateless approach:

```php
<?php
// STATEFUL (violates REST)
session_start();
$_SESSION['page'] = 2;
// A subsequent request relies on this session value.

// STATELESS (RESTful)
// GET /products?page=2&limit=20
// All context is in the URL — no session needed.
$page  = (int) ($_GET['page']  ?? 1);
$limit = (int) ($_GET['limit'] ?? 20);
```

By placing all context in the request itself (URL, headers, body), the server can be horizontally scaled across many machines without worrying about shared session storage.

### 3. Resource-Oriented Design
A resource is any entity the API manages. Resources are identified by nouns in the URL, never verbs.

The following table summarises the standard resource URL patterns:

| URL | Meaning |
|---|---|
| GET /users | List all users |
| GET /users/42 | Retrieve user 42 |
| POST /users | Create a new user |
| PUT /users/42 | Replace user 42 entirely |
| PATCH /users/42 | Partially update user 42 |
| DELETE /users/42 | Remove user 42 |

### 4. HATEOAS
Responses embed links to related resources, enabling clients to navigate the API without hard-coding URLs.

Here is a response that includes hypermedia links:

```php
<?php
declare(strict_types=1);

$user = [
    'id'   => 42,
    'name' => 'Alice Johnson',
    '_links' => [
        'self'   => ['href' => '/users/42'],
        'orders' => ['href' => '/users/42/orders'],
        'avatar' => ['href' => '/users/42/avatar'],
    ],
];

header('Content-Type: application/json');
echo json_encode($user, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT);
```

The `_links` section tells the client exactly where to go next, reducing coupling between the client code and the API's URL structure.

### 5. PHP 8.5 and REST
PHP 8.5 introduces the pipe operator (`|>`), which lets you chain transformations in a readable left-to-right style. This is especially handy when building response payloads:

```php
<?php
declare(strict_types=1);

// PHP 8.5 pipe operator for transforming a resource before responding
function addLinks(array $user): array {
    $user['_links'] = [
        'self'   => ['href' => "/users/{$user['id']}"],
        'orders' => ['href' => "/users/{$user['id']}/orders"],
    ];
    return $user;
}

function toJson(array $data): string {
    return json_encode($data, JSON_THROW_ON_ERROR | JSON_UNESCAPED_SLASHES);
}

$response = ['id' => 42, 'name' => 'Alice']
    |> addLinks(...)
    |> toJson(...);

// $response is a JSON string ready to send
echo $response;
```

The pipe operator makes each transformation step explicit and easy to read from left to right.

## Common Pitfalls
1. **Using verbs in URLs** — Writing `/getUsers` or `/createUser` instead of `GET /users` and `POST /users`. REST relies on HTTP methods as verbs; the URL should only contain nouns.
2. **Storing client state on the server** — Relying on `$_SESSION` to carry context between API calls breaks statelessness. Pass all context through request parameters, headers, or the body.

## Best Practices
1. **Use plural nouns for collections** — `/users` (not `/user`) for the collection endpoint. The singular form is reserved for a single resource: `/users/42`.
2. **Nest resources sparingly** — One level of nesting (`/users/42/orders`) is fine. Deeper nesting (`/users/42/orders/7/items/3/reviews`) becomes hard to maintain; prefer flat URLs with query filters instead.

## Summary
- REST is an architectural style defined by six constraints: client-server, statelessness, cacheability, uniform interface, layered system, and optional code-on-demand.
- Resources are identified by noun-based URLs and manipulated via standard HTTP methods.
- Statelessness means every request carries all the information the server needs.
- HATEOAS embeds navigation links in responses so clients can discover the API.
- PHP 8.5's pipe operator provides a clean way to chain data transformations when building responses.

## Code Examples

**A minimal RESTful endpoint showing statelessness, resource URLs, and HATEOAS links in a single PHP script**

```php
<?php
declare(strict_types=1);

// A minimal RESTful resource endpoint demonstrating
// statelessness, resource-based URLs, and HATEOAS links.

$method = $_SERVER['REQUEST_METHOD'];
$uri    = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// Simulated data store
$users = [
    1 => ['id' => 1, 'name' => 'Alice', 'email' => 'alice@example.com'],
    2 => ['id' => 2, 'name' => 'Bob',   'email' => 'bob@example.com'],
];

header('Content-Type: application/json; charset=utf-8');

if ($method === 'GET' && $uri === '/users') {
    // List all users with HATEOAS links
    $result = array_map(function (array $user): array {
        $user['_links'] = [
            'self' => ['href' => "/users/{$user['id']}"],
        ];
        return $user;
    }, array_values($users));

    http_response_code(200);
    echo json_encode(['data' => $result], JSON_UNESCAPED_SLASHES);
} elseif ($method === 'GET' && preg_match('#^/users/(\d+)$#', $uri, $m)) {
    $id = (int) $m[1];
    if (!isset($users[$id])) {
        http_response_code(404);
        echo json_encode(['error' => 'User not found']);
    } else {
        $user = $users[$id];
        $user['_links'] = [
            'self'   => ['href' => "/users/{$id}"],
            'orders' => ['href' => "/users/{$id}/orders"],
        ];
        http_response_code(200);
        echo json_encode(['data' => $user], JSON_UNESCAPED_SLASHES);
    }
} else {
    http_response_code(404);
    echo json_encode(['error' => 'Not found']);
}
?>
```


## Resources

- [PHP Built-in Web Server](https://www.php.net/manual/en/features.commandline.webserver.php) — Official PHP docs for the built-in development server, useful for testing REST APIs locally
- [json_encode — PHP Manual](https://www.php.net/manual/en/function.json-encode.php) — PHP reference for encoding data as JSON responses

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*