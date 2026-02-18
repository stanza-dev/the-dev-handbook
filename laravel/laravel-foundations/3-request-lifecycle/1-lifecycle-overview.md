---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-request-lifecycle-overview"
---

# The Laravel Request Lifecycle

Understanding how Laravel processes a request helps you write better applications and debug issues effectively. Let's trace a request from browser to response.

## The Journey of a Request

```
┌───────────────────────────────────────────────────────────────┐
│                        HTTP Request                            │
│                   GET /users?page=1                            │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│ 1. Entry Point: public/index.php                              │
│    - Loads Composer autoloader                                │
│    - Creates Application instance                             │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│ 2. HTTP Kernel                                                 │
│    - Loads configuration                                       │
│    - Registers service providers                               │
│    - Boots service providers                                   │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│ 3. Middleware Pipeline                                         │
│    - Global middleware (HTTPS, maintenance mode)              │
│    - Route middleware (auth, throttle)                        │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│ 4. Router                                                      │
│    - Matches URL to route                                     │
│    - Resolves controller/closure                              │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│ 5. Controller/Action                                           │
│    - Handles business logic                                   │
│    - Returns response                                         │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│ 6. Response                                                    │
│    - Passes back through middleware                           │
│    - Sent to browser                                          │
└───────────────────────────────────────────────────────────────┘
```

## Step 1: Entry Point

Every request enters through `public/index.php`:

```php
<?php

use Illuminate\Http\Request;

define('LARAVEL_START', microtime(true));

// Register the Composer autoloader
require __DIR__.'/../vendor/autoload.php';

// Bootstrap Laravel and handle the request
(require_once __DIR__.'/../bootstrap/app.php')
    ->handleRequest(Request::capture());
```

The `bootstrap/app.php` file creates the Application:

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Configure middleware here
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // Configure exception handling here
    })->create();
```

## Step 2: The HTTP Kernel

The HTTP kernel bootstraps the application:

1. **Loads environment** (.env file)
2. **Loads configuration** (config/*.php)
3. **Registers service providers** (register() methods)
4. **Boots service providers** (boot() methods)

## Step 3: Middleware Pipeline

Middleware filters requests before they reach your code:

```php
// Global middleware - runs on every request
$middleware = [
    TrustProxies::class,
    PreventRequestsDuringMaintenance::class,
    ValidatePostSize::class,
    TrimStrings::class,
    ConvertEmptyStringsToNull::class,
];

// Web middleware group
$middlewareGroups = [
    'web' => [
        EncryptCookies::class,
        AddQueuedCookiesToResponse::class,
        StartSession::class,
        ShareErrorsFromSession::class,
        VerifyCsrfToken::class,
        SubstituteBindings::class,
    ],
];
```

Middleware executes in order:

```
Request → M1 → M2 → M3 → Controller → M3 → M2 → M1 → Response
```

## Step 4: Routing

The router matches the URL to a route:

```php
// routes/web.php
Route::get('/users', [UserController::class, 'index']);

// Request: GET /users
// Matched: UserController@index
```

Routing also runs route-specific middleware:

```php
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('auth');  // Only authenticated users
```

## Step 5: Controller/Action

The controller handles the request:

```php
class UserController extends Controller
{
    public function index()
    {
        $users = User::paginate(15);

        return view('users.index', ['users' => $users]);
    }
}
```

Dependencies are automatically injected:

```php
public function store(StoreUserRequest $request)  // Auto-validated!
{
    User::create($request->validated());
    return redirect()->route('users.index');
}
```

## Step 6: Response

The response travels back through middleware:

```php
// Middleware can modify responses
public function handle($request, Closure $next)
{
    $response = $next($request);  // Get response from controller

    // Modify response
    $response->header('X-Custom-Header', 'Value');

    return $response;
}
```

Finally, the response is sent to the browser.

## Resources

- [Request Lifecycle](https://laravel.com/docs/12.x/lifecycle) — Official documentation on Laravel's request lifecycle

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*