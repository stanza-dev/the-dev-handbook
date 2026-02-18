---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-understanding-middleware"
---

# Understanding Middleware

Middleware provides a mechanism for filtering HTTP requests entering your application. Think of middleware as a series of "layers" that requests must pass through before reaching your application.

## What is Middleware?

Middleware acts like a gatekeeper:

```
                    Request
                       │
                       ▼
               ┌───────────────┐
               │  Middleware 1 │  Check if authenticated
               └───────────────┘
                       │
                       ▼
               ┌───────────────┐
               │  Middleware 2 │  Verify CSRF token
               └───────────────┘
                       │
                       ▼
               ┌───────────────┐
               │  Middleware 3 │  Log request
               └───────────────┘
                       │
                       ▼
               ┌───────────────┐
               │  Controller   │  Handle request
               └───────────────┘
                       │
                       ▼
                   Response
```

Common uses for middleware:

- **Authentication**: Is the user logged in?
- **Authorization**: Can the user access this resource?
- **CSRF Protection**: Is this a legitimate form submission?
- **Rate Limiting**: Has this user made too many requests?
- **Logging**: Record request information
- **CORS**: Add cross-origin headers

## Creating Middleware

```bash
php artisan make:middleware EnsureUserIsAdmin
```

This creates `app/Http/Middleware/EnsureUserIsAdmin.php`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsAdmin
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->isAdmin()) {
            return redirect('home');
        }

        return $next($request);
    }
}
```

## Middleware Flow

The `$next` closure passes the request to the next layer:

```php
public function handle(Request $request, Closure $next): Response
{
    // BEFORE: Code here runs before the controller
    $startTime = microtime(true);

    // Pass to next middleware (eventually reaching controller)
    $response = $next($request);

    // AFTER: Code here runs after the controller
    $duration = microtime(true) - $startTime;
    Log::info("Request took {$duration} seconds");

    return $response;
}
```

## Before vs After Middleware

### Before Middleware

Runs before the controller:

```php
public function handle(Request $request, Closure $next): Response
{
    // Perform action BEFORE request is handled
    if ($this->isBlacklisted($request->ip())) {
        abort(403);
    }

    return $next($request);
}
```

### After Middleware

Runs after the controller:

```php
public function handle(Request $request, Closure $next): Response
{
    $response = $next($request);

    // Perform action AFTER request is handled
    $response->header('X-Custom-Header', 'Value');

    return $response;
}
```

## Registering Middleware

Configure middleware in `bootstrap/app.php`:

```php
use App\Http\Middleware\EnsureUserIsAdmin;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        // Alias for route middleware
        $middleware->alias([
            'admin' => EnsureUserIsAdmin::class,
        ]);
    })
    ->create();
```

## Assigning Middleware to Routes

```php
// Single middleware
Route::get('/admin', [AdminController::class, 'index'])
    ->middleware('admin');

// Multiple middleware
Route::get('/admin/users', [AdminController::class, 'users'])
    ->middleware(['auth', 'admin']);

// To a group
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin/dashboard', ...);
    Route::get('/admin/settings', ...);
});

// Inline middleware class
Route::get('/profile', fn() => ...)->middleware(EnsureUserIsAdmin::class);
```

## Excluding Middleware

```php
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', ...);

    // Exclude auth for this route
    Route::get('/public-page', ...)->withoutMiddleware(['auth']);
});
```

## Global Middleware

Run on every request:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->append(LogRequests::class);  // Add to end
    $middleware->prepend(StartTimer::class);  // Add to beginning
})
```

## Middleware Groups

Laravel includes two default groups:

```php
// The 'web' group (sessions, cookies, CSRF)
$middleware->group('web', [
    EncryptCookies::class,
    AddQueuedCookiesToResponse::class,
    StartSession::class,
    ShareErrorsFromSession::class,
    VerifyCsrfToken::class,
    SubstituteBindings::class,
]);

// The 'api' group (stateless)
$middleware->group('api', [
    'throttle:api',
    SubstituteBindings::class,
]);
```

## Resources

- [Middleware](https://laravel.com/docs/12.x/middleware) — Official Laravel middleware documentation

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*