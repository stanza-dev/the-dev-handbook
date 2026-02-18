---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-http-responses"
---

# Creating HTTP Responses

Laravel provides several ways to return responses from your routes and controllers. The framework automatically converts various return types into proper HTTP responses.

## Basic Responses

### Strings

Return a string and Laravel wraps it in a response:

```php
Route::get('/hello', function () {
    return 'Hello World';
});
// Response: 200 OK, Content-Type: text/html
```

### Arrays and JSON

Arrays and Eloquent models are automatically converted to JSON:

```php
Route::get('/user', function () {
    return ['name' => 'John', 'email' => 'john@example.com'];
});
// Response: 200 OK, Content-Type: application/json
// {"name":"John","email":"john@example.com"}

Route::get('/user/{user}', function (User $user) {
    return $user;  // Eloquent model to JSON
});
```

### Response Objects

For full control, use the `response()` helper:

```php
return response('Hello World', 200)
    ->header('Content-Type', 'text/plain');

// With multiple headers
return response($content)
    ->header('Content-Type', 'text/plain')
    ->header('X-Custom-Header', 'Value')
    ->withHeaders([
        'X-Header-One' => 'Value',
        'X-Header-Two' => 'Value',
    ]);
```

### Setting Cookies

```php
return response('Hello')
    ->cookie('name', 'value', $minutes);

// Advanced cookie options
return response('Hello')
    ->cookie('name', 'value', $minutes, $path, $domain, $secure, $httpOnly);
```

## JSON Responses

Explicitly return JSON:

```php
return response()->json([
    'name' => 'John',
    'status' => 'success',
]);

// With custom status code
return response()->json(['error' => 'Not found'], 404);

// JSONP response
return response()
    ->json(['name' => 'John'])
    ->withCallback($request->input('callback'));
```

## View Responses

Return a Blade view:

```php
// Using view() helper
return view('welcome');

// With data
return view('users.show', ['user' => $user]);

// With response customization
return response()
    ->view('users.show', ['user' => $user], 200)
    ->header('Content-Type', 'text/html');
```

## Redirects

Redirect to another URL:

```php
// Basic redirect
return redirect('/home');

// To a named route
return redirect()->route('login');

// With route parameters
return redirect()->route('users.show', ['id' => 1]);

// To a controller action
return redirect()->action([UserController::class, 'index']);

// Back to previous page
return back();
return back()->withInput();  // Keep form input

// To external URL
return redirect()->away('https://google.com');
```

### Redirects with Flash Data

```php
// Flash a message to the session
return redirect('/home')->with('status', 'Profile updated!');

// In the view:
@if (session('status'))
    <div class="alert">{{ session('status') }}</div>
@endif

// Flash errors
return back()->withErrors(['email' => 'Invalid email address']);

// Keep old input
return back()->withInput()->withErrors($errors);
```

## Download Responses

Download files:

```php
// Download a file
return response()->download($pathToFile);

// With custom filename
return response()->download($path, 'report.pdf');

// With headers
return response()->download($path, $name, $headers);

// Delete file after download
return response()->download($path)->deleteFileAfterSend();
```

## Stream Responses

Stream content directly:

```php
return response()->stream(function () {
    echo "First chunk\n";
    ob_flush();
    flush();
    sleep(1);
    echo "Second chunk\n";
    ob_flush();
    flush();
}, 200, ['Content-Type' => 'text/plain']);
```

## File Responses

Display files inline (not download):

```php
// Display image in browser
return response()->file($pathToFile);

// With headers
return response()->file($path, ['Content-Type' => 'image/png']);
```

## Response Macros

Define custom response types:

```php
// In AppServiceProvider boot()
Response::macro('success', function ($data = null, $message = 'Success') {
    return Response::json([
        'success' => true,
        'message' => $message,
        'data' => $data,
    ]);
});

Response::macro('error', function ($message, $status = 400) {
    return Response::json([
        'success' => false,
        'message' => $message,
    ], $status);
});

// Usage
return response()->success($user, 'User created');
return response()->error('Validation failed', 422);
```

## Resources

- [HTTP Responses](https://laravel.com/docs/12.x/responses) — Official documentation on HTTP responses

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*