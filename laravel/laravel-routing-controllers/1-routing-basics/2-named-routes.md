---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-named-routes"
---

# Named Routes and URL Generation

Named routes allow you to generate URLs or redirects without hardcoding paths. If a URL changes, you only update the route definition—not every link in your application.

## Naming Routes

Use the `name()` method:

```php
Route::get('/users', [UserController::class, 'index'])->name('users.index');
Route::get('/users/{id}', [UserController::class, 'show'])->name('users.show');
Route::post('/users', [UserController::class, 'store'])->name('users.store');
```

## Generating URLs

Use the `route()` helper:

```php
// Generate a URL
$url = route('users.index');  // '/users'

// With parameters
$url = route('users.show', ['id' => 1]);  // '/users/1'

// Multiple parameters
Route::get('/posts/{post}/comments/{comment}', ...)->name('comments.show');
$url = route('comments.show', ['post' => 1, 'comment' => 5]);
// '/posts/1/comments/5'

// Extra parameters become query string
$url = route('users.index', ['page' => 2, 'sort' => 'name']);
// '/users?page=2&sort=name'
```

## Using Named Routes in Blade

```blade
<!-- Link to named route -->
<a href="{{ route('users.index') }}">All Users</a>

<!-- With parameters -->
<a href="{{ route('users.show', ['id' => $user->id]) }}">{{ $user->name }}</a>

<!-- Shorthand with Eloquent model -->
<a href="{{ route('users.show', $user) }}">{{ $user->name }}</a>
```

## Redirecting to Named Routes

```php
// In a controller
return redirect()->route('users.index');

// With parameters
return redirect()->route('users.show', ['id' => $user->id]);

// With flash data
return redirect()->route('users.index')->with('status', 'User created!');
```

## Checking the Current Route

```php
// In a controller
if ($request->routeIs('users.*')) {
    // Current route matches 'users.index', 'users.show', etc.
}

// In Blade
@if (request()->routeIs('users.index'))
    <span>You're on the users page</span>
@endif

// Multiple patterns
@if (request()->routeIs('users.*', 'profile'))
    // Matches any users.* route or 'profile'
@endif
```

## Signed URLs

Create URLs that can't be tampered with:

```php
// Generate a signed URL
$url = URL::signedRoute('unsubscribe', ['user' => 1]);
// '/unsubscribe/1?signature=abc123...'

// Temporary signed URL (expires)
$url = URL::temporarySignedRoute(
    'unsubscribe',
    now()->addMinutes(30),
    ['user' => 1]
);
```

Validate signed URLs:

```php
Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // Process unsubscribe...
})->name('unsubscribe');

// Or use middleware
Route::get('/unsubscribe/{user}', function () {
    // ...
})->name('unsubscribe')->middleware('signed');
```

## Route Groups

Group related routes to share attributes:

```php
// Shared middleware
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/profile', [ProfileController::class, 'show']);
});

// Shared prefix
Route::prefix('admin')->group(function () {
    Route::get('/users', ...);      // /admin/users
    Route::get('/settings', ...);   // /admin/settings
});

// Shared name prefix
Route::name('admin.')->group(function () {
    Route::get('/users', ...)->name('users');   // admin.users
    Route::get('/settings', ...)->name('settings'); // admin.settings
});

// Combined
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminController::class, 'dashboard'])
            ->name('dashboard');  // admin.dashboard at /admin/dashboard
    });
```

## Subdomain Routing

```php
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('/users/{id}', function ($account, $id) {
        return "Account: {$account}, User: {$id}";
    });
});
// shop.myapp.com/users/1 → "Account: shop, User: 1"
```

## Route Model Binding

Automatically inject model instances:

```php
// Laravel automatically finds User by ID
Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
// /users/1 → finds User with id=1
// /users/999 → 404 if not found

// Use a different column
Route::get('/users/{user:slug}', function (User $user) {
    return $user;
});
// /users/john-doe → finds User where slug='john-doe'
```

## Resources

- [Named Routes](https://laravel.com/docs/12.x/routing#named-routes) — Official documentation on named routes

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*