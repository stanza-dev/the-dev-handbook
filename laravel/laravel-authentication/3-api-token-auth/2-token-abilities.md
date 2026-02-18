---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-token-abilities"
---

# Token Abilities and Scopes

Token abilities allow you to limit what actions a token can perform. This is crucial for implementing least-privilege access control.

## Creating Tokens with Abilities

```php
// Token with all abilities
$token = $user->createToken('full-access');
// Abilities: ['*']

// Token with specific abilities
$token = $user->createToken('read-only', [
    'posts:read',
    'comments:read',
]);

// More granular abilities
$token = $user->createToken('post-manager', [
    'posts:read',
    'posts:create',
    'posts:update',
    'posts:delete',
]);
```

## Checking Abilities

### In Controllers

```php
class PostController extends Controller
{
    public function index(Request $request)
    {
        // Check if token has ability
        if (! $request->user()->tokenCan('posts:read')) {
            abort(403, 'Token does not have permission to read posts.');
        }

        return Post::all();
    }

    public function store(Request $request)
    {
        if (! $request->user()->tokenCan('posts:create')) {
            abort(403, 'Token does not have permission to create posts.');
        }

        return Post::create($request->validated());
    }
}
```

### Using Middleware

```php
// Require specific abilities
Route::get('/posts', [PostController::class, 'index'])
    ->middleware(['auth:sanctum', 'ability:posts:read']);

Route::post('/posts', [PostController::class, 'store'])
    ->middleware(['auth:sanctum', 'ability:posts:create']);

// Require any of multiple abilities
Route::put('/posts/{post}', [PostController::class, 'update'])
    ->middleware(['auth:sanctum', 'ability:posts:update,posts:admin']);

// abilities (plural) - requires ALL listed abilities
Route::delete('/posts/{post}', [PostController::class, 'destroy'])
    ->middleware(['auth:sanctum', 'abilities:posts:delete,posts:admin']);
```

### ability vs abilities Middleware

```php
// ability: ANY of the listed abilities
->middleware('ability:read,write')  // Token needs read OR write

// abilities: ALL of the listed abilities
->middleware('abilities:read,write')  // Token needs read AND write
```

## Organizing Abilities

Create a central place for ability definitions:

```php
// app/Enums/TokenAbility.php
<?php

namespace App\Enums;

class TokenAbility
{
    // Post abilities
    public const POSTS_READ = 'posts:read';
    public const POSTS_CREATE = 'posts:create';
    public const POSTS_UPDATE = 'posts:update';
    public const POSTS_DELETE = 'posts:delete';

    // Comment abilities
    public const COMMENTS_READ = 'comments:read';
    public const COMMENTS_CREATE = 'comments:create';
    public const COMMENTS_DELETE = 'comments:delete';

    // User abilities
    public const USERS_READ = 'users:read';
    public const USERS_UPDATE = 'users:update';

    // Admin abilities
    public const ADMIN_ACCESS = 'admin:access';

    // Predefined sets
    public const READ_ONLY = [
        self::POSTS_READ,
        self::COMMENTS_READ,
        self::USERS_READ,
    ];

    public const STANDARD_USER = [
        self::POSTS_READ,
        self::POSTS_CREATE,
        self::COMMENTS_READ,
        self::COMMENTS_CREATE,
        self::USERS_READ,
        self::USERS_UPDATE,
    ];

    public const ADMIN = ['*'];
}
```

Usage:

```php
use App\Enums\TokenAbility;

// Create tokens with predefined sets
$token = $user->createToken('mobile', TokenAbility::STANDARD_USER);

// Check abilities
if ($request->user()->tokenCan(TokenAbility::POSTS_CREATE)) {
    // ...
}
```

## Token Expiration

Configure in `config/sanctum.php`:

```php
'expiration' => 60 * 24 * 7,  // 7 days in minutes

// Or null for no expiration
'expiration' => null,
```

### Prune Expired Tokens

```bash
php artisan sanctum:prune-expired

# Prune tokens expired more than 24 hours ago
php artisan sanctum:prune-expired --hours=24
```

Schedule in `routes/console.php`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

## Managing User Tokens

```php
// Get all tokens for user
$tokens = $user->tokens;

// Get current token
$currentToken = $request->user()->currentAccessToken();

// Revoke current token (logout)
$request->user()->currentAccessToken()->delete();

// Revoke a specific token by ID
$user->tokens()->where('id', $tokenId)->delete();

// Revoke all tokens (logout everywhere)
$user->tokens()->delete();

// Revoke all tokens except current
$user->tokens()->where('id', '!=', $currentToken->id)->delete();
```

## Token Management UI

```php
// Controller for token management
class TokenController extends Controller
{
    public function index(Request $request)
    {
        return $request->user()->tokens;
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'abilities' => 'array',
            'abilities.*' => 'string',
        ]);

        $token = $request->user()->createToken(
            $validated['name'],
            $validated['abilities'] ?? ['*']
        );

        return response()->json([
            'token' => $token->plainTextToken,
            'abilities' => $token->accessToken->abilities,
        ], 201);
    }

    public function destroy(Request $request, $tokenId)
    {
        $request->user()->tokens()->where('id', $tokenId)->delete();

        return response()->json(['message' => 'Token revoked']);
    }
}
```

## Resources

- [Token Abilities](https://laravel.com/docs/12.x/sanctum#token-abilities) — Official documentation on Sanctum token abilities

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*