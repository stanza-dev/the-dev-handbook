---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-sanctum-fundamentals"
---

# Laravel Sanctum Fundamentals

Laravel Sanctum provides a featherweight authentication system for SPAs (Single Page Applications), mobile applications, and simple token-based APIs. It's the go-to solution for most Laravel API authentication needs.

## When to Use Sanctum

| Use Case | Sanctum Feature |
|----------|------------------|
| First-party SPA (React, Vue, Angular) | Cookie-based session auth |
| Mobile applications | API token authentication |
| Third-party API consumers | API token authentication |
| Simple API authentication | Either approach works |

## Sanctum vs Passport

```
┌─────────────────────────────────────────────────────────────────┐
│                     SANCTUM                                     │
├─────────────────────────────────────────────────────────────────┤
│ ✓ Lightweight                                                   │
│ ✓ Simple API tokens                                            │
│ ✓ SPA authentication with cookies                              │
│ ✓ No OAuth complexity                                          │
│ ✓ Perfect for first-party apps                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     PASSPORT                                     │
├─────────────────────────────────────────────────────────────────┤
│ ✓ Full OAuth2 server implementation                            │
│ ✓ Authorization codes, refresh tokens                          │
│ ✓ Client credentials grant                                     │
│ ✓ Perfect for third-party API access                          │
│ ✗ More complex setup                                           │
└─────────────────────────────────────────────────────────────────┘
```

**Rule of thumb**: Use Sanctum unless you specifically need OAuth2.

## Installing Sanctum

Sanctum comes pre-installed in new Laravel applications. If you need to install it:

```bash
composer require laravel/sanctum

php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

php artisan migrate
```

This creates the `personal_access_tokens` table.

## Configuring the User Model

Add the `HasApiTokens` trait to your User model:

```php
<?php

namespace App\Models;

use Laravel\Sanctum\HasApiTokens;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    // ...
}
```

## How Token Authentication Works

```
1. User logs in with credentials
   ┌─────────────────────────────────────────────┐
   │ POST /api/login                             │
   │ { "email": "...", "password": "..." }       │
   └─────────────────────────────────────────────┘
                     │
                     ▼
2. Server validates and creates token
   ┌─────────────────────────────────────────────┐
   │ $user->createToken('device-name')           │
   │ Returns: { "token": "1|abc123..." }         │
   └─────────────────────────────────────────────┘
                     │
                     ▼
3. Client stores token and sends with requests
   ┌─────────────────────────────────────────────┐
   │ GET /api/user                               │
   │ Authorization: Bearer 1|abc123...           │
   └─────────────────────────────────────────────┘
                     │
                     ▼
4. Server validates token and returns data
   ┌─────────────────────────────────────────────┐
   │ auth('sanctum')->user() returns User        │
   └─────────────────────────────────────────────┘
```

## Creating Tokens

```php
// Simple token
$token = $user->createToken('device-name');
$plainTextToken = $token->plainTextToken;  // "1|abc123..."

// Token with abilities (permissions)
$token = $user->createToken('device-name', ['posts:read', 'posts:write']);

// Token with expiration (configured in config/sanctum.php)
```

## Token Storage

Tokens are stored hashed in the `personal_access_tokens` table:

```sql
| id | tokenable_type | tokenable_id | name      | token (hashed) | abilities   |
|----|----------------|--------------|-----------|----------------|-------------|
| 1  | App\Models\User| 5            | mobile-app| 7f3a8b2c...    | ["*"]       |
| 2  | App\Models\User| 5            | web-app   | 9e4d1f5a...    | ["read"]    |
```

## Authentication Endpoint Example

```php
// routes/api.php
use App\Http\Controllers\Api\AuthController;

Route::post('/login', [AuthController::class, 'login']);
Route::post('/register', [AuthController::class, 'register']);

Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', [AuthController::class, 'user']);
    Route::post('/logout', [AuthController::class, 'logout']);
});
```

```php
// app/Http/Controllers/Api/AuthController.php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function login(Request $request)
    {
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

        return response()->json([
            'user' => $user,
            'token' => $user->createToken($request->device_name)->plainTextToken,
        ]);
    }

    public function user(Request $request)
    {
        return $request->user();
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json(['message' => 'Logged out']);
    }
}
```

## Protecting Routes

```php
// Protect with Sanctum
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/posts', [PostController::class, 'index']);
    Route::post('/posts', [PostController::class, 'store']);
});
```

## Resources

- [Laravel Sanctum](https://laravel.com/docs/12.x/sanctum) — Official Laravel Sanctum documentation

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*