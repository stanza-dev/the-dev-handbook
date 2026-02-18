---
source_course: "php-api-development"
source_lesson: "php-api-development-basic-router"
---

# Building a Router

A router maps HTTP requests to handler functions based on URL patterns and HTTP methods.

## Simple Router

```php
<?php
class Router {
    private array $routes = [];
    
    public function get(string $path, callable $handler): void {
        $this->addRoute('GET', $path, $handler);
    }
    
    public function post(string $path, callable $handler): void {
        $this->addRoute('POST', $path, $handler);
    }
    
    public function put(string $path, callable $handler): void {
        $this->addRoute('PUT', $path, $handler);
    }
    
    public function patch(string $path, callable $handler): void {
        $this->addRoute('PATCH', $path, $handler);
    }
    
    public function delete(string $path, callable $handler): void {
        $this->addRoute('DELETE', $path, $handler);
    }
    
    private function addRoute(string $method, string $path, callable $handler): void {
        $this->routes[] = [
            'method' => $method,
            'path' => $path,
            'handler' => $handler,
        ];
    }
    
    public function dispatch(string $method, string $uri): mixed {
        foreach ($this->routes as $route) {
            if ($route['method'] !== $method) {
                continue;
            }
            
            $params = $this->matchPath($route['path'], $uri);
            
            if ($params !== false) {
                return call_user_func_array($route['handler'], $params);
            }
        }
        
        http_response_code(404);
        return ['error' => 'Not Found'];
    }
    
    private function matchPath(string $routePath, string $uri): array|false {
        // Convert /users/{id} to regex
        $pattern = preg_replace('/\{([a-z]+)\}/', '(?P<$1>[^/]+)', $routePath);
        $pattern = '#^' . $pattern . '$#';
        
        if (preg_match($pattern, $uri, $matches)) {
            // Return only named captures
            return array_filter($matches, 'is_string', ARRAY_FILTER_USE_KEY);
        }
        
        return false;
    }
}
```

## Usage

```php
<?php
$router = new Router();

$router->get('/users', function() {
    return ['users' => User::all()];
});

$router->get('/users/{id}', function(string $id) {
    $user = User::find((int) $id);
    if (!$user) {
        http_response_code(404);
        return ['error' => 'User not found'];
    }
    return ['user' => $user->toArray()];
});

$router->post('/users', function() {
    $data = json_decode(file_get_contents('php://input'), true);
    $user = User::create($data);
    http_response_code(201);
    return ['user' => $user->toArray()];
});

$router->delete('/users/{id}', function(string $id) {
    $user = User::find((int) $id);
    if ($user) {
        $user->delete();
    }
    http_response_code(204);
    return null;
});

// Dispatch
$method = $_SERVER['REQUEST_METHOD'];
$uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

header('Content-Type: application/json');
$result = $router->dispatch($method, $uri);

if ($result !== null) {
    echo json_encode($result);
}
```

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*