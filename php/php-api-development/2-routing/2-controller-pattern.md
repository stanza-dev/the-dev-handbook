---
source_course: "php-api-development"
source_lesson: "php-api-development-controller-pattern"
---

# The Controller Pattern

## Introduction
As your API grows, stuffing all handler logic into anonymous functions becomes unmanageable. The controller pattern solves this by organising related handlers into classes where each method corresponds to one API action. This lesson shows how to separate routing from business logic using controllers.

## Key Concepts
- **Controller**: A class that groups the handler methods for a single resource (e.g., `UserController` handles all `/users` operations).
- **Action method**: A public method on a controller that handles one specific request (e.g., `index`, `show`, `store`, `update`, `destroy`).
- **Separation of concerns**: Routing decides *which* controller to call; the controller decides *what* to do with the request.

## Real World Context
Every major PHP framework uses the controller pattern. Laravel's resource controllers map seven standard methods to routes automatically. Even in micro-frameworks like Slim, grouping handlers into controller classes keeps your codebase navigable as it grows from ten endpoints to a hundred.

## Deep Dive

### Base Controller
Start with an abstract base that provides common helpers every controller needs:

```php
<?php
declare(strict_types=1);

abstract class Controller {
    /**
     * Send a JSON response and terminate the script.
     */
    protected function json(mixed $data, int $status = 200): never {
        http_response_code($status);
        header('Content-Type: application/json; charset=utf-8');
        echo json_encode($data, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE);
        exit;
    }

    /**
     * Read and decode the JSON request body.
     */
    protected function input(): array {
        $raw = file_get_contents('php://input');
        if ($raw === '' || $raw === false) {
            return [];
        }
        return json_decode($raw, associative: true, flags: JSON_THROW_ON_ERROR);
    }

    /**
     * Return a 404 Not Found response.
     */
    protected function notFound(string $message = 'Resource not found'): never {
        $this->json(['error' => ['message' => $message]], 404);
    }
}
```

The `never` return type signals that these methods terminate execution. This helps static analysers flag any code that accidentally follows a response call.

### Resource Controller
Now create a concrete controller for the `users` resource. Each public method maps to one HTTP endpoint:

```php
<?php
declare(strict_types=1);

class UserController extends Controller {
    // In a real app this would be injected via constructor
    private array $users = [
        1 => ['id' => 1, 'name' => 'Alice', 'email' => 'alice@example.com'],
        2 => ['id' => 2, 'name' => 'Bob',   'email' => 'bob@example.com'],
    ];

    /** GET /users */
    public function index(array $params): never {
        $this->json(['data' => array_values($this->users)]);
    }

    /** GET /users/{id} */
    public function show(array $params): never {
        $id   = (int) $params['id'];
        $user = $this->users[$id] ?? null;

        if ($user === null) {
            $this->notFound('User not found');
        }

        $this->json(['data' => $user]);
    }

    /** POST /users */
    public function store(array $params): never {
        $body = $this->input();
        $newId = max(array_keys($this->users)) + 1;

        $user = [
            'id'    => $newId,
            'name'  => $body['name'] ?? '',
            'email' => $body['email'] ?? '',
        ];

        $this->json(['data' => $user], 201);
    }

    /** DELETE /users/{id} */
    public function destroy(array $params): never {
        http_response_code(204);
        exit;
    }
}
```

Each method has a single, clear responsibility. The controller knows nothing about URL matching — that is the router's job.

### Wiring Controllers to the Router
Connect the router to controller methods using class-method pairs:

```php
<?php
declare(strict_types=1);

$router = new Router();

$router->get('/users',       function (array $p): void { (new UserController())->index($p); });
$router->get('/users/{id}',  function (array $p): void { (new UserController())->show($p); });
$router->post('/users',      function (array $p): void { (new UserController())->store($p); });
$router->delete('/users/{id}', function (array $p): void { (new UserController())->destroy($p); });

header('Content-Type: application/json');
$router->dispatch(
    $_SERVER['REQUEST_METHOD'],
    parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH)
);
```

The thin closure wrapping keeps the router decoupled from the controller instantiation. In a real application you would use a dependency injection container to create controllers.

### Naming Conventions
The standard CRUD action names make your codebase predictable:

| Method | URL | Action | Purpose |
|---|---|---|---|
| GET | /users | `index` | List all |
| GET | /users/{id} | `show` | Show one |
| POST | /users | `store` | Create |
| PUT/PATCH | /users/{id} | `update` | Update |
| DELETE | /users/{id} | `destroy` | Delete |

Using these names consistently means any developer joining the project knows exactly where to look.

## Common Pitfalls
1. **Fat controllers** — Putting database queries, validation, email sending, and logging all inside a single controller method. Controllers should delegate to services. Keep action methods under 20 lines of orchestration code.
2. **Instantiating dependencies manually** — Writing `new UserRepository()` inside every method duplicates setup code and makes testing hard. Pass dependencies through the constructor so they can be mocked in tests.

## Best Practices
1. **One controller per resource** — `UserController` handles `/users`, `ProductController` handles `/products`. Do not create a single `ApiController` that handles everything.
2. **Keep controllers thin** — The controller's job is to read the request, call a service, and return a response. Business logic belongs in service classes, not controllers.

## Summary
- Controllers group related request handlers into classes with standard action methods (`index`, `show`, `store`, `update`, `destroy`).
- An abstract base controller provides shared helpers like `json()`, `input()`, and `notFound()`.
- The router delegates to controllers but does not depend on their internal logic.
- Use consistent naming conventions so developers can navigate the codebase by convention.
- Keep controllers thin: they orchestrate, services compute.

## Code Examples

**Base controller with json/input/notFound helpers and a ProductController implementing standard CRUD actions**

```php
<?php
declare(strict_types=1);

// A base controller and a resource controller working together.

abstract class Controller {
    protected function json(mixed $data, int $status = 200): never {
        http_response_code($status);
        header('Content-Type: application/json; charset=utf-8');
        echo json_encode($data, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE);
        exit;
    }

    protected function input(): array {
        $raw = file_get_contents('php://input');
        return ($raw !== '' && $raw !== false)
            ? json_decode($raw, true, 512, JSON_THROW_ON_ERROR)
            : [];
    }

    protected function notFound(string $msg = 'Not found'): never {
        $this->json(['error' => ['message' => $msg]], 404);
    }
}

class ProductController extends Controller {
    private array $products = [
        1 => ['id' => 1, 'name' => 'Mechanical Keyboard', 'price' => 129.99],
        2 => ['id' => 2, 'name' => '4K Monitor',          'price' => 449.00],
    ];

    /** GET /products — list all */
    public function index(array $params): never {
        $this->json(['data' => array_values($this->products)]);
    }

    /** GET /products/{id} — show one */
    public function show(array $params): never {
        $id = (int) $params['id'];
        $product = $this->products[$id] ?? null;

        if ($product === null) {
            $this->notFound('Product not found');
        }

        $this->json(['data' => $product]);
    }

    /** POST /products — create */
    public function store(array $params): never {
        $body    = $this->input();
        $newId   = max(array_keys($this->products)) + 1;
        $product = [
            'id'    => $newId,
            'name'  => $body['name']  ?? '',
            'price' => (float) ($body['price'] ?? 0),
        ];

        // 201 Created with Location header
        header("Location: /products/{$newId}");
        $this->json(['data' => $product], 201);
    }

    /** DELETE /products/{id} — remove */
    public function destroy(array $params): never {
        // 204 No Content — empty response
        http_response_code(204);
        exit;
    }
}
?>
```


## Resources

- [call_user_func — PHP Manual](https://www.php.net/manual/en/function.call-user-func.php) — PHP reference for dynamically invoking functions and methods, used by routers to call controller actions

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*