---
source_course: "laravel-api-development"
source_lesson: "laravel-api-development-sanctum-introduction"
---

# Introduction to Laravel Sanctum

Laravel Sanctum provides a lightweight authentication system for SPAs, mobile applications, and simple token-based APIs.

## When to Use Sanctum

| Use Case | Sanctum Feature |
|----------|------------------|
| First-party SPA (React, Vue) | Cookie-based authentication |
| Mobile apps | API token authentication |
| Third-party API consumers | API token authentication |
| Simple API authentication | Either approach |

## Installing Sanctum

Sanctum comes installed with Laravel by default:

```bash
# If not installed
composer require laravel/sanctum

# Publish config and migrations
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

# Run migrations
php artisan migrate
```

This creates the `personal_access_tokens` table.

## API Token Authentication

For mobile apps and third-party API consumers:

### 1. Add the Trait to User Model

```php
<?php

namespace App\Models;

use Laravel\Sanctum\HasApiTokens;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use HasApiTokens;

    // ...
}
```

### 2. Create a Token Issuance Endpoint

```php
// routes/api.php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

Route::post('/login', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    // Create and return the token
    return response()->json([
        'token' => $user->createToken($request->device_name)->plainTextToken,
        'user' => $user,
    ]);
});
```

### 3. Protect Routes

```php
// routes/api.php
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', function (Request $request) {
        return $request->user();
    });

    Route::apiResource('posts', PostController::class);
});
```

### 4. Using the Token

```bash
# Include token in Authorization header
curl http://localhost:8000/api/user \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Accept: application/json"
```

## Token Abilities (Permissions)

Limit what tokens can do:

```php
// Create token with abilities
$token = $user->createToken('api-token', ['posts:read']);

// Create token with all abilities
$token = $user->createToken('admin-token', ['*']);

// Check abilities in routes/controller
Route::get('/posts', function (Request $request) {
    if (! $request->user()->tokenCan('posts:read')) {
        abort(403);
    }

    return Post::all();
});

// Using middleware
Route::get('/posts', [PostController::class, 'index'])
    ->middleware(['auth:sanctum', 'ability:posts:read']);

Route::post('/posts', [PostController::class, 'store'])
    ->middleware(['auth:sanctum', 'ability:posts:write']);
```

## Revoking Tokens

```php
// Revoke current token
$request->user()->currentAccessToken()->delete();

// Revoke all tokens
$user->tokens()->delete();

// Revoke specific token
$user->tokens()->where('id', $tokenId)->delete();

// Logout endpoint
Route::post('/logout', function (Request $request) {
    $request->user()->currentAccessToken()->delete();

    return response()->json(['message' => 'Logged out']);
})->middleware('auth:sanctum');
```

## Token Expiration

Configure in `config/sanctum.php`:

```php
// Tokens expire after 1 week
'expiration' => 60 * 24 * 7,  // minutes

// Or null for no expiration
'expiration' => null,
```

Prune expired tokens:

```bash
php artisan sanctum:prune-expired

# In scheduled commands
$schedule->command('sanctum:prune-expired --hours=24')->daily();
```

## Complete Authentication Controller

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|string|min:8|confirmed',
            'device_name' => 'required|string',
        ]);

        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        return response()->json([
            'token' => $user->createToken($validated['device_name'])->plainTextToken,
            'user' => $user,
        ], 201);
    }

    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
            'device_name' => 'required|string',
        ]);

        $user = User::where('email', $request->email)->first();

        if (! $user || ! Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        return response()->json([
            'token' => $user->createToken($request->device_name)->plainTextToken,
            'user' => $user,
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json(['message' => 'Successfully logged out']);
    }

    public function user(Request $request)
    {
        return response()->json($request->user());
    }
}
```

Routes:

```php
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->group(function () {
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::get('/user', [AuthController::class, 'user']);
});
```

## Resources

- [Laravel Sanctum](https://laravel.com/docs/12.x/sanctum) — Official Sanctum documentation

---

> 📘 *This lesson is part of the [Laravel API Development](https://stanza.dev/courses/laravel-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*