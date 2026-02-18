---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-built-in-middleware"
---

# Laravel's Built-in Middleware

Laravel includes several middleware out of the box that handle common web application needs. Understanding these helps you build secure, robust applications.

## Authentication Middleware

The `auth` middleware redirects guests to the login page:

```php
// Require authentication
Route::get('/dashboard', DashboardController::class)
    ->middleware('auth');

// Specify a guard
Route::get('/admin', AdminController::class)
    ->middleware('auth:admin');

// Multiple guards
Route::get('/api/user', fn() => auth()->user())
    ->middleware('auth:sanctum,api');
```

## Guest Middleware

The `guest` middleware redirects authenticated users:

```php
// Only for guests (not logged in)
Route::get('/login', [LoginController::class, 'showLoginForm'])
    ->middleware('guest');

Route::get('/register', [RegisterController::class, 'showRegistrationForm'])
    ->middleware('guest');
```

## Throttle Middleware (Rate Limiting)

Prevent abuse with rate limiting:

```php
// 60 requests per minute
Route::get('/api/data', DataController::class)
    ->middleware('throttle:60,1');

// Named rate limiter
Route::get('/api/data', DataController::class)
    ->middleware('throttle:api');
```

Define custom rate limiters in `AppServiceProvider`:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()?->isPremium()
            ? Limit::none()
            : Limit::perMinute(10)->by($request->user()->id);
    });
}
```

## CSRF Protection

The `VerifyCsrfToken` middleware protects against cross-site request forgery:

```blade
<!-- Include CSRF token in forms -->
<form method="POST" action="/users">
    @csrf  <!-- Adds hidden _token field -->
    <input type="text" name="name">
    <button type="submit">Submit</button>
</form>
```

Exclude routes from CSRF protection:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
        'webhook/*',
    ]);
})
```

## Signed URL Middleware

Validate signed URLs:

```php
// Generate signed URL
$url = URL::signedRoute('unsubscribe', ['user' => 1]);

// Validate with middleware
Route::get('/unsubscribe/{user}', [UnsubscribeController::class, 'handle'])
    ->name('unsubscribe')
    ->middleware('signed');
```

## Password Confirmation Middleware

Require password re-entry for sensitive actions:

```php
Route::get('/settings/security', [SecurityController::class, 'index'])
    ->middleware(['auth', 'password.confirm']);

// User must confirm password within last 3 hours (default)
```

## Verified Email Middleware

```php
Route::get('/dashboard', DashboardController::class)
    ->middleware(['auth', 'verified']);

// Unverified users are redirected to email verification notice
```

## Maintenance Mode Middleware

`PreventRequestsDuringMaintenance` redirects to maintenance page:

```bash
# Enable maintenance mode
php artisan down

# Allow specific IPs
php artisan down --allow=192.168.1.1

# With secret bypass
php artisan down --secret="bypass-token"
# Visit /bypass-token to get a cookie that bypasses maintenance
```

## Cache Headers Middleware

Add caching headers to responses:

```php
Route::get('/static-page', StaticController::class)
    ->middleware('cache.headers:public;max_age=2628000;etag');
```

## Practical Example: Combining Middleware

```php
// API routes with multiple protections
Route::prefix('api/v1')
    ->middleware(['api', 'auth:sanctum', 'throttle:api'])
    ->group(function () {
        Route::get('/user', fn(Request $request) => $request->user());

        Route::apiResource('posts', PostController::class);

        // Premium features with different throttle
        Route::middleware('throttle:premium')
            ->group(function () {
                Route::post('/analyze', AnalyzeController::class);
            });
    });

// Admin routes with multiple guards
Route::prefix('admin')
    ->middleware(['auth', 'verified', 'password.confirm'])
    ->group(function () {
        Route::get('/dashboard', [AdminController::class, 'dashboard']);

        // Extra protection for dangerous actions
        Route::delete('/users/{user}', [AdminController::class, 'destroyUser'])
            ->middleware('can:delete-users');
    });
```

## Resources

- [CSRF Protection](https://laravel.com/docs/12.x/csrf) — Official CSRF protection documentation
- [Rate Limiting](https://laravel.com/docs/12.x/routing#rate-limiting) — Official rate limiting documentation

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*