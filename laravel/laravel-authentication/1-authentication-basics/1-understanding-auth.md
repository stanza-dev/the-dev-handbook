---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-understanding-auth"
---

# Understanding Laravel Authentication

Authentication is the process of verifying who a user is. Laravel provides a robust authentication system out of the box, handling everything from login forms to API tokens.

## Authentication vs Authorization

```
┌─────────────────────────────────────────────────────────────────┐
│                      AUTHENTICATION                              │
│               "Who are you?"                                    │
│                                                                 │
│    Login credentials → Verify identity → Create session         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      AUTHORIZATION                               │
│               "What can you do?"                                │
│                                                                 │
│    Check permissions → Allow/Deny access → Execute action       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

- **Authentication**: Verifying identity (login/logout)
- **Authorization**: Controlling access (permissions, roles)

## Laravel's Authentication Architecture

### Guards

Guards define **how** users are authenticated for each request:

```php
// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',      // Uses cookies/session
        'provider' => 'users',
    ],
    'api' => [
        'driver' => 'sanctum',      // Uses tokens
        'provider' => 'users',
    ],
],
```

### Providers

Providers define **where** user data comes from:

```php
'providers' => [
    'users' => [
        'driver' => 'eloquent',     // Uses Eloquent model
        'model' => App\Models\User::class,
    ],
    // Or use database directly
    'legacy_users' => [
        'driver' => 'database',
        'table' => 'users',
    ],
],
```

## The Default User Model

Laravel includes a User model with authentication traits:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',  // Auto-hash on set
        ];
    }
}
```

## Basic Authentication Flow

```
1. User submits login form (email + password)
                 │
                 ▼
2. Auth::attempt() verifies credentials
                 │
                 ▼
3. If valid → Create session + Set cookie
   If invalid → Return error
                 │
                 ▼
4. Subsequent requests:
   Cookie sent → Session retrieved → User loaded
```

## Accessing the Authenticated User

```php
// Using the Auth facade
use Illuminate\Support\Facades\Auth;

$user = Auth::user();      // Get user instance or null
$id = Auth::id();          // Get user ID or null
$check = Auth::check();    // Is someone logged in?
$guest = Auth::guest();    // Is no one logged in?

// Using the auth() helper
$user = auth()->user();
$id = auth()->id();

// Using the request
public function index(Request $request)
{
    $user = $request->user();  // Same as Auth::user()
}

// In Blade templates
@auth
    <p>Welcome, {{ auth()->user()->name }}!</p>
@endauth

@guest
    <a href="/login">Login</a>
@endguest
```

## Password Hashing

Never store plain-text passwords:

```php
use Illuminate\Support\Facades\Hash;

// Hash a password
$hashed = Hash::make('password');

// Verify a password
if (Hash::check('password', $hashedPassword)) {
    // Passwords match
}

// With the 'hashed' cast (Laravel 10+)
class User extends Authenticatable
{
    protected function casts(): array
    {
        return [
            'password' => 'hashed',  // Auto-hash on assignment
        ];
    }
}

// Now this auto-hashes:
$user->password = 'newpassword';  // Stored as hash
```

## Starter Kits

Laravel offers starter kits for complete auth scaffolding:

### Laravel Breeze

```bash
composer require laravel/breeze --dev
php artisan breeze:install

# Choose your stack:
# - Blade
# - Livewire
# - React
# - Vue
# - API only

npm install && npm run dev
php artisan migrate
```

Breeze provides:
- Login/Register/Logout
- Password reset
- Email verification
- Profile management

### Laravel Jetstream

More features for larger applications:
- Team management
- Two-factor authentication
- API tokens
- Session management

## Resources

- [Authentication](https://laravel.com/docs/12.x/authentication) — Official Laravel authentication documentation
- [Laravel Breeze](https://laravel.com/docs/12.x/starter-kits#laravel-breeze) — Authentication starter kit documentation

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*