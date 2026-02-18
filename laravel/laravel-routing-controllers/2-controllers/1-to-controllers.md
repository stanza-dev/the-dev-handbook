---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-introduction-to-controllers"
---

# Introduction to Controllers

Controllers group related request handling logic into a single class. Instead of defining all logic in route closures, controllers help organize your application.

## Why Controllers?

```php
// ❌ Route closures work but don't scale
Route::get('/users', function () {
    $users = User::all();
    // 50 lines of logic...
    return view('users.index', compact('users'));
});

Route::post('/users', function (Request $request) {
    // Another 50 lines...
});

// ✅ Controllers organize related logic
Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);
```

## Creating Controllers

Use Artisan to generate controllers:

```bash
# Basic controller
php artisan make:controller UserController

# Resource controller (with CRUD methods)
php artisan make:controller PostController --resource

# API resource controller (no create/edit)
php artisan make:controller API/ProductController --api

# Single action controller
php artisan make:controller ShowProfile --invokable
```

## Basic Controller Structure

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Display a listing of users.
     */
    public function index(): View
    {
        $users = User::all();

        return view('users.index', ['users' => $users]);
    }

    /**
     * Display the specified user.
     */
    public function show(User $user): View
    {
        return view('users.show', ['user' => $user]);
    }

    /**
     * Store a newly created user.
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
        ]);

        $user = User::create($validated);

        return redirect()->route('users.show', $user);
    }
}
```

## Defining Routes for Controllers

```php
use App\Http\Controllers\UserController;

// Single method
Route::get('/users', [UserController::class, 'index']);

// Multiple methods
Route::get('/users', [UserController::class, 'index'])->name('users.index');
Route::get('/users/{user}', [UserController::class, 'show'])->name('users.show');
Route::post('/users', [UserController::class, 'store'])->name('users.store');
```

## Single Action Controllers

For controllers with only one method, use `__invoke`:

```php
<?php

namespace App\Http\Controllers;

class ShowProfile extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request)
    {
        return view('profile', [
            'user' => $request->user(),
        ]);
    }
}
```

Route definition is simpler:

```php
// No method needed!
Route::get('/profile', ShowProfile::class);
```

## Dependency Injection

Laravel's service container resolves controller dependencies:

```php
<?php

namespace App\Http\Controllers;

use App\Services\UserService;

class UserController extends Controller
{
    /**
     * Constructor injection
     */
    public function __construct(
        private UserService $userService
    ) {}

    public function index()
    {
        $users = $this->userService->getAllActive();

        return view('users.index', ['users' => $users]);
    }

    /**
     * Method injection
     */
    public function store(Request $request, UserService $service)
    {
        $user = $service->create($request->validated());

        return redirect()->route('users.show', $user);
    }
}
```

## Controller Middleware

Apply middleware to all or specific controller methods:

```php
// In routes
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('auth');

// In controller constructor
class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('admin')->only('destroy');
        $this->middleware('throttle:60,1')->except(['index', 'show']);
    }
}

// Modern approach: use attributes (PHP 8+)
#[Middleware('auth')]
class UserController extends Controller
{
    // All methods require auth
}
```

## Resources

- [Controllers](https://laravel.com/docs/12.x/controllers) — Official Laravel controllers documentation

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*