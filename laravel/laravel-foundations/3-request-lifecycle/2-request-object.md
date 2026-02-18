---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-http-request-object"
---

# Working with HTTP Requests

The `Illuminate\Http\Request` class provides an object-oriented way to access all HTTP request data. Laravel automatically injects this into your controllers.

## Accessing the Request

Type-hint the Request class in controller methods:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;

class UserController extends Controller
{
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // Store the user...

        return redirect('/users');
    }
}
```

With route parameters:

```php
public function update(Request $request, string $id): RedirectResponse
{
    // $request has the HTTP data
    // $id has the route parameter

    return redirect('/users');
}
```

## Request Path and URL

```php
// URL: https://example.com/users?page=2

$request->path();           // 'users'
$request->url();            // 'https://example.com/users'
$request->fullUrl();        // 'https://example.com/users?page=2'
$request->fullUrlWithQuery(['sort' => 'name']);
                            // 'https://example.com/users?page=2&sort=name'

// Check the path
$request->is('users/*');    // true if path matches pattern
$request->routeIs('users.index');  // true if named route matches
```

## Request Method

```php
$request->method();         // 'GET', 'POST', etc.
$request->isMethod('post'); // true if POST

// Method spoofing for forms
// <input type="hidden" name="_method" value="PUT">
```

## Retrieving Input

Get data from any source (query string, form data, JSON):

```php
// Get a single value
$name = $request->input('name');

// With default
$name = $request->input('name', 'Guest');

// Nested data: products[0][name]
$name = $request->input('products.0.name');

// All nested data
$names = $request->input('products.*.name');

// Get all input
$all = $request->all();

// Get only specific keys
$data = $request->only(['name', 'email']);

// Get all except specific keys
$data = $request->except(['password']);
```

### Query String vs Form Data

```php
// Only query string: /users?search=john
$search = $request->query('search');

// All query string parameters
$query = $request->query();

// Only POST data
$name = $request->post('name');
```

### JSON Requests

```php
// For Content-Type: application/json
$name = $request->input('user.name');

// Or access the raw JSON
$data = $request->json()->all();
```

## Dynamic Properties

Access input as properties:

```php
$name = $request->name;  // Same as $request->input('name')
$email = $request->email;

// Route parameters take precedence over input
```

## Checking for Input

```php
// Check if present (even if empty)
if ($request->has('name')) {
    // 'name' key exists
}

// Check multiple keys
if ($request->has(['name', 'email'])) {
    // Both exist
}

// Check if any are present
if ($request->hasAny(['name', 'nickname'])) {
    // At least one exists
}

// Check if present AND not empty
if ($request->filled('name')) {
    // Has a non-empty value
}

// Check if missing or empty
if ($request->missing('name')) {
    // 'name' is not in the request
}
```

## Files

Handle uploaded files:

```php
// Get a file
$file = $request->file('photo');

// Or use dynamic property
$file = $request->photo;

// Check if file was uploaded
if ($request->hasFile('photo')) {
    // File exists
}

// Validate and store
if ($request->file('photo')->isValid()) {
    $path = $request->photo->store('photos');  // Store in storage/app/photos
    $path = $request->photo->storeAs('photos', 'filename.jpg');
}

// Get file info
$name = $request->photo->getClientOriginalName();
$extension = $request->photo->extension();
$size = $request->photo->getSize();
```

## Headers and Cookies

```php
// Get a header
$token = $request->header('X-API-Token');

// With default
$contentType = $request->header('Content-Type', 'text/html');

// Get a cookie
$remember = $request->cookie('remember_token');

// Bearer token from Authorization header
$token = $request->bearerToken();
```

## Request Information

```php
$request->ip();              // Client IP address
$request->userAgent();       // User agent string
$request->server('SERVER_PROTOCOL');  // Server variable

// Content negotiation
$request->accepts(['text/html', 'application/json']);
$request->wantsJson();       // Expects JSON response?
```

## Resources

- [HTTP Requests](https://laravel.com/docs/12.x/requests) — Official documentation on HTTP requests

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*