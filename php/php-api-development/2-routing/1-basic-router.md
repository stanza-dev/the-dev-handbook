---
source_course: "php-api-development"
source_lesson: "php-api-development-basic-router"
---

# Building a Simple Router

## Introduction
A router is the front door of your API. It examines the incoming HTTP method and URL, decides which piece of code should handle the request, and extracts any dynamic parameters from the path. In this lesson you will build a simple router from scratch using core PHP functions.

## Key Concepts
- **Router**: A component that maps HTTP method + URL path combinations to handler functions.
- **Route parameter**: A dynamic placeholder in a URL pattern (e.g., `{id}` in `/users/{id}`) that captures a value from the actual request URL.
- **REQUEST_URI**: The PHP superglobal `$_SERVER['REQUEST_URI']` that contains the path and query string of the incoming request.
- **Dispatching**: The process of finding the matching route and calling its handler function.

## Real World Context
Every PHP framework — Laravel, Symfony, Slim — has a router at its core. Even if you use a framework in production, understanding how routing works under the hood helps you debug unexpected 404s, understand middleware execution order, and build custom solutions when a framework's router does not fit your needs.

## Deep Dive

### Step 1: Parse the Request
The first thing a router needs is the HTTP method and a clean URL path:

```php
<?php
declare(strict_types=1);

// Extract the method and path from the request
$method = $_SERVER['REQUEST_METHOD'];                    // e.g. "GET"
$uri    = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH); // e.g. "/users/42"

// Remove trailing slash for consistency (except root "/")
if ($uri !== '/') {
    $uri = rtrim($uri, '/');
}
```

Using `parse_url()` strips the query string so `/users?page=2` becomes `/users`. The trailing-slash trim ensures `/users/` and `/users` match the same route.

### Step 2: Register Routes
A route is a combination of an HTTP method, a URL pattern, and a handler. Store them in a simple array:

```php
<?php
declare(strict_types=1);

class Router {
    /** @var array<int, array{method: string, pattern: string, handler: callable}> */
    private array $routes = [];

    public function get(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'GET', 'pattern' => $pattern, 'handler' => $handler];
    }

    public function post(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'POST', 'pattern' => $pattern, 'handler' => $handler];
    }

    public function put(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'PUT', 'pattern' => $pattern, 'handler' => $handler];
    }

    public function delete(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'DELETE', 'pattern' => $pattern, 'handler' => $handler];
    }
}
```

Each public method pushes a new entry onto the `$routes` array with the corresponding HTTP verb.

### Step 3: Match Routes and Extract Parameters
To match a request against registered routes, convert the pattern into a regular expression that captures named groups:

```php
<?php
// Inside the Router class

private function matchRoute(string $pattern, string $uri): array|false {
    // Convert {param} placeholders to named regex groups
    // e.g., "/users/{id}" becomes "#^/users/(?P<id>[^/]+)$#"
    $regex = preg_replace('/\{([a-zA-Z_]+)\}/', '(?P<$1>[^/]+)', $pattern);
    $regex = '#^' . $regex . '$#';

    if (preg_match($regex, $uri, $matches)) {
        // Return only named captures (the parameter values)
        return array_filter($matches, 'is_string', ARRAY_FILTER_USE_KEY);
    }

    return false;
}
```

The `array_filter` with `ARRAY_FILTER_USE_KEY` removes numeric indices from the regex matches, leaving only named parameters like `['id' => '42']`.

### Step 4: Dispatch
Loop through routes, find the first match, and call its handler:

```php
<?php
// Inside the Router class

public function dispatch(string $method, string $uri): void {
    foreach ($this->routes as $route) {
        if ($route['method'] !== $method) {
            continue;
        }

        $params = $this->matchRoute($route['pattern'], $uri);

        if ($params !== false) {
            call_user_func($route['handler'], $params);
            return;
        }
    }

    // No route matched
    http_response_code(404);
    header('Content-Type: application/json');
    echo json_encode(['error' => 'Route not found']);
}
```

The dispatch method iterates every registered route. When it finds a match on both method and path, it calls the handler with the extracted parameters and returns. If no route matches, the client receives a 404.

### Putting It All Together
Here is how you use the complete router:

```php
<?php
declare(strict_types=1);

$router = new Router();

$router->get('/users', function (array $params): void {
    echo json_encode(['data' => [['id' => 1, 'name' => 'Alice']]]);
});

$router->get('/users/{id}', function (array $params): void {
    $userId = (int) $params['id'];
    echo json_encode(['data' => ['id' => $userId, 'name' => 'Alice']]);
});

$router->post('/users', function (array $params): void {
    $body = json_decode(file_get_contents('php://input'), true);
    http_response_code(201);
    echo json_encode(['data' => ['id' => 99, 'name' => $body['name'] ?? 'Unknown']]);
});

// Dispatch the current request
header('Content-Type: application/json');
$method = $_SERVER['REQUEST_METHOD'];
$uri    = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
$router->dispatch($method, $uri);
```

The router is now a self-contained component that can be tested and extended independently.

## Common Pitfalls
1. **Forgetting to strip the query string** — If you match against `$_SERVER['REQUEST_URI']` directly, a request to `/users?page=2` will not match the `/users` route. Always use `parse_url($uri, PHP_URL_PATH)` first.
2. **Route ordering matters** — A catch-all pattern like `/users/{id}` placed before `/users/export` will match `/users/export` with `id = 'export'`. Register specific routes before parameterised ones.

## Best Practices
1. **Normalise trailing slashes** — Trim trailing slashes from the URI before matching so `/users/` and `/users` resolve to the same route. This prevents confusing 404 errors.
2. **Return 405 for wrong methods** — If the path matches but the method does not, return `405 Method Not Allowed` with an `Allow` header listing the valid methods. This is more helpful than a generic 404.

## Summary
- A router maps HTTP method + URL path pairs to handler functions.
- Use `parse_url()` to isolate the path from the query string.
- Convert `{param}` placeholders to named regex groups for parameter extraction.
- Dispatch loops through routes and calls the first matching handler.
- Register specific routes before parameterised ones to avoid false matches.

## Code Examples

**Complete minimal router with parameter extraction, dispatching, and 404 handling for a /products resource**

```php
<?php
declare(strict_types=1);

/**
 * A complete, minimal PHP router.
 * Supports GET, POST, PUT, PATCH, DELETE with {param} placeholders.
 */
class Router {
    /** @var array<int, array{method: string, pattern: string, handler: callable}> */
    private array $routes = [];

    public function get(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'GET', 'pattern' => $pattern, 'handler' => $handler];
    }

    public function post(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'POST', 'pattern' => $pattern, 'handler' => $handler];
    }

    public function put(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'PUT', 'pattern' => $pattern, 'handler' => $handler];
    }

    public function patch(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'PATCH', 'pattern' => $pattern, 'handler' => $handler];
    }

    public function delete(string $pattern, callable $handler): void {
        $this->routes[] = ['method' => 'DELETE', 'pattern' => $pattern, 'handler' => $handler];
    }

    public function dispatch(string $method, string $uri): void {
        $uri = rtrim($uri, '/') ?: '/';

        foreach ($this->routes as $route) {
            if ($route['method'] !== $method) {
                continue;
            }

            $params = $this->match($route['pattern'], $uri);
            if ($params !== false) {
                call_user_func($route['handler'], $params);
                return;
            }
        }

        http_response_code(404);
        echo json_encode(['error' => 'Route not found']);
    }

    private function match(string $pattern, string $uri): array|false {
        $regex = preg_replace('/\{([a-zA-Z_]+)\}/', '(?P<$1>[^/]+)', $pattern);
        if (preg_match('#^' . $regex . '$#', $uri, $matches)) {
            return array_filter($matches, 'is_string', ARRAY_FILTER_USE_KEY);
        }
        return false;
    }
}

// Example usage
$router = new Router();

$router->get('/products', function (array $params): void {
    echo json_encode(['data' => [['id' => 1, 'name' => 'Keyboard']]]);
});

$router->get('/products/{id}', function (array $params): void {
    echo json_encode(['data' => ['id' => (int) $params['id']]]);
});

$router->post('/products', function (array $params): void {
    $body = json_decode(file_get_contents('php://input'), true);
    http_response_code(201);
    echo json_encode(['data' => $body]);
});

$router->delete('/products/{id}', function (array $params): void {
    http_response_code(204);
});

header('Content-Type: application/json');
$router->dispatch(
    $_SERVER['REQUEST_METHOD'],
    parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH)
);
?>
```


## Resources

- [parse_url — PHP Manual](https://www.php.net/manual/en/function.parse-url.php) — Official PHP reference for extracting URL components
- [preg_match — PHP Manual](https://www.php.net/manual/en/function.preg-match.php) — PHP reference for regex pattern matching, used to extract route parameters

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*