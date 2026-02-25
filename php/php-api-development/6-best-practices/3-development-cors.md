---
source_course: "php-api-development"
source_lesson: "php-api-development-cors"
---

# CORS Configuration

## Introduction
When your JavaScript frontend at `app.example.com` tries to call your API at `api.example.com`, the browser blocks the request by default. This is the Same-Origin Policy in action. Cross-Origin Resource Sharing (CORS) is the mechanism that tells browsers which cross-origin requests to allow.

## Key Concepts
- **Same-Origin Policy**: A browser security feature that prevents JavaScript from making requests to a different origin (protocol + domain + port) than the page it was loaded from.
- **CORS (Cross-Origin Resource Sharing)**: A protocol that uses HTTP headers to tell browsers which cross-origin requests are permitted.
- **Preflight Request**: An automatic `OPTIONS` request the browser sends before certain cross-origin requests to check if the server allows the actual request.
- **Simple Request**: A cross-origin request that uses only GET, HEAD, or POST with standard headers, which does not trigger a preflight.
- **Access-Control Headers**: HTTP headers like `Access-Control-Allow-Origin` and `Access-Control-Allow-Methods` that configure CORS policy.

## Real World Context
Every single-page application (SPA) that communicates with a separate API backend needs CORS configuration. Without it, your React, Vue, or Angular frontend cannot make API calls to your PHP backend if they are on different domains or ports. Misconfigured CORS is one of the most common deployment issues for web APIs.

## Deep Dive

### How CORS Works

CORS uses a handshake between the browser and server. For non-simple requests, the browser sends a preflight `OPTIONS` request first. The server responds with headers indicating what is allowed.

Here is a diagram of the flow:

1. Browser sends `OPTIONS /api/products` with origin and requested method/headers.
2. Server responds with `Access-Control-Allow-*` headers.
3. If allowed, browser sends the actual `POST /api/products` request.
4. Server includes `Access-Control-Allow-Origin` in the response.

### Building CORS Middleware

A CORS middleware intercepts requests and adds the appropriate headers. Here is a complete, configurable CORS middleware:

```php
<?php
class CorsMiddleware
{
    /** @param string[] $allowedOrigins */
    public function __construct(
        private readonly array $allowedOrigins = ['*'],
        private readonly array $allowedMethods = ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
        private readonly array $allowedHeaders = ['Content-Type', 'Authorization', 'X-Requested-With'],
        private readonly int $maxAge = 86400,
        private readonly bool $allowCredentials = false,
    ) {}

    public function handle(callable $next): void
    {
        $origin = $_SERVER['HTTP_ORIGIN'] ?? '';

        if (!$this->isOriginAllowed($origin)) {
            if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
                http_response_code(403);
                exit;
            }
            $next();
            return;
        }

        // Set CORS headers
        header("Access-Control-Allow-Origin: {$origin}");
        header('Access-Control-Allow-Methods: ' . implode(', ', $this->allowedMethods));
        header('Access-Control-Allow-Headers: ' . implode(', ', $this->allowedHeaders));
        header("Access-Control-Max-Age: {$this->maxAge}");

        if ($this->allowCredentials) {
            header('Access-Control-Allow-Credentials: true');
        }

        // Handle preflight
        if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
            http_response_code(204);
            exit;
        }

        $next();
    }

    private function isOriginAllowed(string $origin): bool
    {
        if (in_array('*', $this->allowedOrigins, true)) {
            return true;
        }
        return in_array($origin, $this->allowedOrigins, true);
    }
}
```

The middleware checks the `Origin` header, validates it against the allowed list, and sets the appropriate `Access-Control-*` response headers. Preflight `OPTIONS` requests return a `204 No Content` response immediately without processing the request further.

### Configuring CORS per Environment

Development and production environments typically need different CORS settings. Here is how to configure the middleware based on the environment:

```php
<?php
// Environment-specific CORS configuration
$corsConfig = match ($_ENV['APP_ENV'] ?? 'production') {
    'development' => [
        'origins' => ['http://localhost:3000', 'http://localhost:4200'],
        'credentials' => true,
    ],
    'staging' => [
        'origins' => ['https://staging.myapp.com'],
        'credentials' => true,
    ],
    'production' => [
        'origins' => ['https://myapp.com', 'https://www.myapp.com'],
        'credentials' => true,
    ],
};

$cors = new CorsMiddleware(
    allowedOrigins: $corsConfig['origins'],
    allowCredentials: $corsConfig['credentials'],
);

// Apply to all API routes
$cors->handle(function () use ($router) {
    $router->dispatch($_SERVER['REQUEST_METHOD'], $_SERVER['REQUEST_URI']);
});
```

In development, you allow localhost origins. In production, you lock it down to your exact domains. The `match` expression keeps the configuration clean and readable.

### Exposing Custom Headers

By default, browsers only expose a small set of response headers to JavaScript. If your API returns custom headers (like pagination cursors or rate limit info), you must explicitly expose them:

```php
<?php
// Allow JavaScript to read these custom response headers
header('Access-Control-Expose-Headers: X-Request-Id, X-RateLimit-Remaining, Link');
```

Without this header, `fetch()` in the browser can see the response body but not your custom headers. This is a frequently overlooked CORS detail.

## Common Pitfalls
1. **Using `Access-Control-Allow-Origin: *` with credentials** — Browsers reject this combination. When `Access-Control-Allow-Credentials: true` is set, you must echo back the specific origin from the request, not the wildcard `*`.
2. **Forgetting to handle OPTIONS requests** — If your router does not handle `OPTIONS` method requests, preflight checks fail with a 404 or 405, and the browser blocks all non-simple cross-origin requests.

## Best Practices
1. **Whitelist specific origins** — Avoid `Access-Control-Allow-Origin: *` in production. Maintain an explicit list of allowed origins to prevent unauthorized domains from calling your API.
2. **Cache preflight responses** — Set `Access-Control-Max-Age` to a reasonable value (e.g., 86400 seconds = 24 hours) so browsers do not send a preflight request before every single API call.

## Summary
- The Same-Origin Policy blocks cross-origin requests by default; CORS headers tell browsers which exceptions to allow.
- Preflight `OPTIONS` requests happen automatically for non-simple requests (custom headers, non-GET/POST methods).
- A CORS middleware centralizes header management and handles preflight responses.
- Never use wildcard origins (`*`) with credentials; echo the specific origin instead.
- Use `Access-Control-Expose-Headers` to make custom response headers visible to JavaScript.

## Code Examples

**CORS middleware applied to all API routes — validates origins, sets headers, and handles preflight requests before dispatching to the router**

```php
<?php
// CORS middleware with environment-specific configuration
$cors = new CorsMiddleware(
    allowedOrigins: ['https://myapp.com', 'https://admin.myapp.com'],
    allowedMethods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
    maxAge: 86400,
    allowCredentials: true,
);

// The middleware:
// 1. Validates the Origin header against the allowed list
// 2. Sets Access-Control-Allow-* response headers
// 3. Handles preflight OPTIONS requests with 204 No Content
// 4. Passes non-preflight requests to the next handler
$cors->handle(function () use ($router) {
    $router->dispatch(
        $_SERVER['REQUEST_METHOD'],
        $_SERVER['REQUEST_URI']
    );
});
```


## Resources

- [PHP header — Send HTTP Headers](https://www.php.net/manual/en/function.header.php) — Official PHP documentation for the header() function used to set CORS response headers

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*