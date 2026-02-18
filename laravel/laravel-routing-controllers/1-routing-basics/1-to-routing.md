---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-introduction-to-routing"
---

# Introduction to Routing

Routing is the mechanism that maps URLs to specific code in your application. When a user visits `/users`, how does Laravel know what to display? Routes define this mapping.

## What is a Route?

A route consists of:
1. An **HTTP method** (GET, POST, PUT, DELETE, etc.)
2. A **URI pattern** (`/users`, `/posts/{id}`)
3. An **action** (closure or controller method)

```php
// When someone visits /hello via GET request
Route::get('/hello', function () {
    return 'Hello, World!';
});
```

## Route Files

Laravel organizes routes into separate files:

```
routes/
├── web.php     # Web routes with sessions, CSRF
├── api.php     # API routes (stateless)
├── console.php # Artisan commands
└── channels.php # Broadcast channels
```

### routes/web.php

For traditional web pages with sessions and CSRF protection:

```php
// routes/web.php
Route::get('/', function () {
    return view('welcome');
});

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth']);
```

### routes/api.php

For stateless API endpoints. Routes are automatically prefixed with `/api`:

```php
// routes/api.php
// This becomes /api/users
Route::get('/users', function () {
    return User::all();
});
```

## Basic Route Definition

### GET Requests

For retrieving data:

```php
// Visit /users to see all users
Route::get('/users', function () {
    return view('users.index', ['users' => User::all()]);
});

// Visit /about to see the about page
Route::get('/about', function () {
    return view('about');
});
```

### POST Requests

For submitting data:

```php
Route::post('/users', function (Request $request) {
    User::create($request->all());
    return redirect('/users');
});
```

### Other HTTP Methods

```php
// Update a resource
Route::put('/users/{id}', function ($id) {
    // Update user
});

Route::patch('/users/{id}', function ($id) {
    // Partial update
});

// Delete a resource
Route::delete('/users/{id}', function ($id) {
    // Delete user
});

// Match multiple methods
Route::match(['get', 'post'], '/form', function () {
    // Handle GET or POST
});

// Match ANY method
Route::any('/webhook', function () {
    // Handle any HTTP method
});
```

## Route Parameters

Capture segments of the URI:

```php
// Required parameter
Route::get('/users/{id}', function (string $id) {
    return "User ID: {$id}";
});
// /users/1 → "User ID: 1"
// /users/42 → "User ID: 42"
// /users → 404 Not Found

// Optional parameter
Route::get('/users/{name?}', function (?string $name = 'Guest') {
    return "Hello, {$name}";
});
// /users → "Hello, Guest"
// /users/John → "Hello, John"

// Multiple parameters
Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
    return "Post {$postId}, Comment {$commentId}";
});
```

## Route Constraints

Limit what parameters can match:

```php
// Only numeric IDs
Route::get('/users/{id}', function ($id) {
    return User::findOrFail($id);
})->where('id', '[0-9]+');

// Only alphabetic names
Route::get('/users/{name}', function ($name) {
    return "Hello, {$name}";
})->where('name', '[A-Za-z]+');

// Multiple constraints
Route::get('/posts/{id}/{slug}', function ($id, $slug) {
    // ...
})->where(['id' => '[0-9]+', 'slug' => '[a-z-]+']);

// Helper methods
Route::get('/users/{id}', $action)->whereNumber('id');
Route::get('/users/{name}', $action)->whereAlpha('name');
Route::get('/users/{name}', $action)->whereAlphaNumeric('name');
Route::get('/users/{uuid}', $action)->whereUuid('uuid');
```

## Global Constraints

Apply constraints to all routes in `AppServiceProvider`:

```php
public function boot(): void
{
    Route::pattern('id', '[0-9]+');
}

// Now all {id} parameters must be numeric
```

## Resources

- [Routing](https://laravel.com/docs/12.x/routing) — Official Laravel routing documentation

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*