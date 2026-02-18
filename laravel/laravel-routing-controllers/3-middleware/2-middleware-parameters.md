---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-middleware-parameters"
---

# Middleware Parameters and Responses

Middleware can accept parameters and return different types of responses. This makes middleware highly flexible and reusable.

## Middleware Parameters

Pass parameters to middleware after a colon:

```php
Route::get('/admin', [AdminController::class, 'index'])
    ->middleware('role:admin');

Route::get('/posts', [PostController::class, 'index'])
    ->middleware('role:editor,admin');  // Multiple values
```

Receive parameters in the middleware:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     * @param  string  ...$roles  // Variadic parameter for multiple roles
     */
    public function handle(Request $request, Closure $next, string ...$roles)
    {
        if (! $request->user() || ! $request->user()->hasAnyRole($roles)) {
            abort(403, 'Unauthorized action.');
        }

        return $next($request);
    }
}
```

Register with an alias:

```php
$middleware->alias([
    'role' => EnsureUserHasRole::class,
]);
```

## Middleware with Multiple Parameters

```php
// Route definition
Route::get('/api/data', [ApiController::class, 'data'])
    ->middleware('throttle:60,1');  // 60 requests per 1 minute

// Middleware
public function handle(Request $request, Closure $next, int $maxAttempts, int $decayMinutes)
{
    // $maxAttempts = 60
    // $decayMinutes = 1
}
```

## Common Middleware Responses

### Redirect Response

```php
public function handle(Request $request, Closure $next)
{
    if (! $request->user()?->hasVerifiedEmail()) {
        return redirect()->route('verification.notice');
    }

    return $next($request);
}
```

### Abort Response

```php
public function handle(Request $request, Closure $next)
{
    if ($request->user()?->isBanned()) {
        abort(403, 'Your account has been banned.');
    }

    return $next($request);
}
```

### JSON Response (for APIs)

```php
public function handle(Request $request, Closure $next)
{
    if (! $request->user()?->isSubscribed()) {
        return response()->json([
            'error' => 'Subscription required',
            'message' => 'Please upgrade your plan to access this feature.',
        ], 403);
    }

    return $next($request);
}
```

## Modifying Requests

Add data to the request:

```php
public function handle(Request $request, Closure $next)
{
    // Add data to request
    $request->merge(['injected_value' => 'something']);

    // Or set an attribute
    $request->attributes->set('start_time', microtime(true));

    return $next($request);
}
```

## Modifying Responses

```php
public function handle(Request $request, Closure $next)
{
    $response = $next($request);

    // Add headers
    $response->header('X-Frame-Options', 'DENY');
    $response->header('X-Content-Type-Options', 'nosniff');

    return $response;
}
```

## Terminable Middleware

Run code after the response is sent to the browser:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class LogSlowRequests
{
    public function handle(Request $request, Closure $next)
    {
        $request->attributes->set('start_time', microtime(true));

        return $next($request);
    }

    /**
     * Handle tasks after the response has been sent.
     */
    public function terminate(Request $request, $response): void
    {
        $duration = microtime(true) - $request->attributes->get('start_time');

        if ($duration > 1.0) {  // Longer than 1 second
            Log::warning('Slow request', [
                'url' => $request->fullUrl(),
                'duration' => round($duration, 2) . 's',
            ]);
        }
    }
}
```

Terminable middleware must be registered as a singleton:

```php
// In AppServiceProvider
$this->app->singleton(LogSlowRequests::class);
```

## Practical Example: API Version Middleware

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class ApiVersion
{
    public function handle(Request $request, Closure $next, string $version)
    {
        // Check Accept header for version
        $acceptHeader = $request->header('Accept', '');

        if (str_contains($acceptHeader, "application/vnd.api.v{$version}")) {
            return $next($request);
        }

        // Check URL for version
        if ($request->segment(2) === "v{$version}") {
            return $next($request);
        }

        return response()->json([
            'error' => 'API version mismatch',
            'expected' => $version,
        ], 400);
    }
}

// Usage
Route::prefix('api/v1')
    ->middleware('api.version:1')
    ->group(function () {
        // v1 routes
    });
```

## Resources

- [Middleware Parameters](https://laravel.com/docs/12.x/middleware#middleware-parameters) — Official documentation on middleware parameters

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*