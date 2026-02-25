---
source_course: "php-api-development"
source_lesson: "php-api-development-auth-middleware"
---

# Middleware Authentication Pattern

## Introduction
Copying authentication logic into every route handler is tedious, error-prone, and a maintenance nightmare. The middleware pattern solves this by extracting authentication into a reusable layer that runs before your route handlers. In this lesson you will build a composable authentication middleware in PHP that protects routes, rejects unauthorized requests early, and makes the authenticated user available to downstream handlers.

## Key Concepts
- **Middleware**: A function that intercepts a request before it reaches the route handler. It can modify the request, return a response early, or pass control to the next middleware.
- **Early Return**: The practice of immediately sending a 401 response when authentication fails, preventing the request from reaching the handler.
- **Request Decoration**: Attaching extra data (like the authenticated user) to the request object so downstream code can use it without repeating the lookup.
- **Middleware Stack**: An ordered list of middleware functions that a request passes through, one by one, before reaching the final handler.

## Real World Context
Every PHP framework — Laravel, Symfony, Slim, Mezzio — uses middleware for authentication. Even if you are building a minimal API without a framework, the middleware pattern keeps your route handlers focused on business logic instead of security boilerplate. You will see this pattern in every professional PHP codebase.

## Deep Dive

### The Problem: Repeated Auth Checks

Without middleware, every protected route duplicates the same authentication code:

```php
<?php
// Route 1: Get profile
$router->get('/profile', function (Request $request) use ($jwt) {
    $user = authenticateRequest($request, $jwt);
    if (!$user) {
        http_response_code(401);
        return ['error' => 'Unauthorized'];
    }
    return $userService->getProfile($user->id);
});

// Route 2: Update settings — same auth check copied again
$router->put('/settings', function (Request $request) use ($jwt) {
    $user = authenticateRequest($request, $jwt);
    if (!$user) {
        http_response_code(401);
        return ['error' => 'Unauthorized'];
    }
    return $settingsService->update($user->id, $request->json());
});
```

This duplication violates DRY and makes it easy to forget the check on a new route. Middleware eliminates this problem.

### Building an Authentication Middleware

A middleware is a callable that takes the request and a `$next` callable. It either returns a response (blocking the request) or calls `$next` to pass control forward.

Here is a complete JWT authentication middleware:

```php
<?php
use Firebase\JWT\JWT;
use Firebase\JWT\Key;

class JwtAuthMiddleware {
    public function __construct(
        private readonly string $secretKey,
    ) {}

    public function __invoke(Request $request, callable $next): array {
        $authHeader = $request->header('Authorization');

        // Early return: no header at all
        if ($authHeader === null) {
            http_response_code(401);
            return ['error' => 'Missing Authorization header'];
        }

        // Early return: malformed header
        if (!preg_match('/^Bearer\s+(\S+)$/', $authHeader, $matches)) {
            http_response_code(401);
            return ['error' => 'Invalid Authorization format. Expected: Bearer <token>'];
        }

        try {
            $decoded = JWT::decode($matches[1], new Key($this->secretKey, 'HS256'));
        } catch (\Exception $e) {
            // Early return: invalid or expired token
            http_response_code(401);
            return ['error' => 'Invalid or expired token'];
        }

        // Attach the authenticated user to the request
        $request->setAuthUser($decoded);

        // Authentication passed — continue to the next middleware or handler
        return $next($request);
    }
}
```

The early return pattern is key: each failure condition exits immediately with a 401 response. Only valid requests reach the `$next` call.

### Registering Middleware on Routes

Now you can protect routes without repeating any auth logic:

```php
<?php
$authMiddleware = new JwtAuthMiddleware($_ENV['JWT_SECRET']);

// Protected routes — middleware runs first, handler only runs if auth passes
$router->get('/profile', function (Request $request) {
    $user = $request->getAuthUser();
    return $userService->getProfile($user->sub);
})->middleware($authMiddleware);

$router->put('/settings', function (Request $request) {
    $user = $request->getAuthUser();
    return $settingsService->update($user->sub, $request->json());
})->middleware($authMiddleware);

// Public routes — no middleware, no auth required
$router->get('/health', function () {
    return ['status' => 'ok'];
});
```

The route handlers are now clean and focused. They simply call `$request->getAuthUser()` to get the authenticated user, trusting that the middleware already validated the token.

### Composing Multiple Middleware

You can stack middleware for layered security. For example, authenticate first, then check for admin role:

```php
<?php
class RequireRoleMiddleware {
    public function __construct(
        private readonly string $requiredRole,
    ) {}

    public function __invoke(Request $request, callable $next): array {
        $user = $request->getAuthUser();

        // Auth middleware already ran, so $user is guaranteed to exist
        if ($user->role !== $this->requiredRole) {
            http_response_code(403);
            return ['error' => 'Insufficient permissions'];
        }

        return $next($request);
    }
}

// Stack: JWT auth first, then role check
$adminOnly = new RequireRoleMiddleware('admin');

$router->delete('/users/{id}', function (Request $request, string $id) {
    return $userService->delete((int) $id);
})->middleware($authMiddleware)->middleware($adminOnly);
```

The middleware stack runs in order: first `JwtAuthMiddleware` verifies the token, then `RequireRoleMiddleware` checks the role. If either fails, the handler never executes.

### Route Group Protection

For APIs with many protected routes, you can apply middleware to an entire group:

```php
<?php
// All routes in this group require authentication
$router->group('/api/v1', function ($group) {
    $group->get('/profile', [ProfileController::class, 'show']);
    $group->put('/profile', [ProfileController::class, 'update']);
    $group->get('/orders', [OrderController::class, 'index']);
    $group->post('/orders', [OrderController::class, 'store']);
})->middleware($authMiddleware);

// Public routes outside the group
$router->post('/auth/login', [AuthController::class, 'login']);
$router->post('/auth/register', [AuthController::class, 'register']);
```

Grouping keeps your route definitions organized and ensures you never forget to protect a route that should be private.

## Common Pitfalls
1. **Forgetting to call `$next()`** — If your middleware does not call `$next($request)` on the success path, the request will never reach the handler. Every middleware must either return an error response or call the next middleware.
2. **Applying auth middleware to login routes** — The login endpoint creates tokens, so it cannot require a token. Make sure public routes like `/auth/login` and `/auth/register` are excluded from authentication middleware.

## Best Practices
1. **Use early returns for clarity** — Check each failure condition and return immediately. This avoids deep nesting and makes the middleware easy to read from top to bottom.
2. **Separate authentication from authorization middleware** — Keep "who are you" (JWT verification) and "what can you do" (role/permission checks) in different middleware classes. This lets you mix and match: some routes need auth only, others need auth + admin.

## Summary
- Middleware intercepts requests before they reach route handlers, enabling reusable authentication logic.
- The early return pattern sends 401/403 responses immediately on failure, preventing unauthorized access to handlers.
- Request decoration attaches the authenticated user to the request object for downstream use.
- Middleware can be stacked (auth, then role check) and applied to individual routes or entire route groups.
- Always separate authentication middleware from authorization middleware for maximum flexibility.

## Code Examples

**JWT authentication middleware with early return on failure and request decoration for the authenticated user**

```php
<?php
class JwtAuthMiddleware {
    public function __construct(
        private readonly string $secretKey,
    ) {}

    public function __invoke(Request $request, callable $next): array {
        $authHeader = $request->header('Authorization');

        if (!preg_match('/^Bearer\s+(\S+)$/', $authHeader ?? '', $matches)) {
            http_response_code(401);
            return ['error' => 'Unauthorized'];
        }

        try {
            $decoded = JWT::decode($matches[1], new Key($this->secretKey, 'HS256'));
        } catch (\Exception $e) {
            http_response_code(401);
            return ['error' => 'Invalid token'];
        }

        $request->setAuthUser($decoded);
        return $next($request); // Pass to handler
    }
}

// Apply to routes
$router->get('/profile', $handler)->middleware($authMiddleware);
```


## Resources

- [PHP Callable type](https://www.php.net/manual/en/language.types.callable.php) — PHP documentation on callable types used for middleware next functions
- [PHP preg_match function](https://www.php.net/manual/en/function.preg-match.php) — Regular expression matching used to parse the Authorization header

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*