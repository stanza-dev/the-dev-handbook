---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-passport-oauth-server"
---

# Building an OAuth2 Server with Passport

Laravel Passport provides a full OAuth2 server implementation. Use it when you need to provide OAuth2 API access to third-party applications.

## When to Use Passport

- Third-party developers need to integrate with your API
- You need refresh tokens
- You need OAuth2 authorization codes
- You're building an API that others will consume

## Installation

```bash
composer require laravel/passport

php artisan migrate

php artisan passport:install
```

Add the trait to User model:

```php
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
}
```

Configure auth guard in `config/auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

## OAuth2 Grant Types

Passport supports all OAuth2 grant types:

### 1. Authorization Code Grant (Most Common)

For web applications where users authorize third-party access:

```
User → Third-Party App → Your Auth Server → User Authorizes → 
 Third-Party gets code → Exchanges for token
```

### 2. Personal Access Tokens

For users to generate their own API tokens:

```php
$token = $user->createToken('Token Name')->accessToken;
```

### 3. Password Grant (First-Party Apps)

For your own mobile/SPA apps:

```bash
php artisan passport:client --password
```

```php
// Exchange username/password for token
POST /oauth/token
{
    "grant_type": "password",
    "client_id": "client-id",
    "client_secret": "client-secret",
    "username": "user@example.com",
    "password": "password",
    "scope": ""
}
```

### 4. Client Credentials Grant

For machine-to-machine authentication:

```bash
php artisan passport:client --client
```

```php
POST /oauth/token
{
    "grant_type": "client_credentials",
    "client_id": "client-id",
    "client_secret": "client-secret",
    "scope": "read-data"
}
```

## Creating OAuth Clients

```bash
# Interactive client creation
php artisan passport:client

# Specific grant types
php artisan passport:client --password
php artisan passport:client --client
php artisan passport:client --personal
```

## Authorization Code Flow

### Step 1: Redirect to Authorization

```
GET /oauth/authorize?
    client_id=CLIENT_ID&
    redirect_uri=https://app.com/callback&
    response_type=code&
    scope=read-posts+write-posts&
    state=RANDOM_STATE
```

### Step 2: User Authorizes (Shows Laravel's Built-in UI)

### Step 3: Callback with Code

```
https://app.com/callback?code=AUTH_CODE&state=RANDOM_STATE
```

### Step 4: Exchange Code for Token

```php
POST /oauth/token
{
    "grant_type": "authorization_code",
    "client_id": "CLIENT_ID",
    "client_secret": "CLIENT_SECRET",
    "redirect_uri": "https://app.com/callback",
    "code": "AUTH_CODE"
}
```

Response:

```json
{
    "token_type": "Bearer",
    "expires_in": 31536000,
    "access_token": "eyJ0eXAiOiJKV1Q...",
    "refresh_token": "def50200..."
}
```

## Defining Scopes

```php
// In AppServiceProvider or AuthServiceProvider
use Laravel\Passport\Passport;

public function boot(): void
{
    Passport::tokensCan([
        'read-posts' => 'Read posts',
        'write-posts' => 'Create and update posts',
        'delete-posts' => 'Delete posts',
        'read-profile' => 'Read user profile',
        'update-profile' => 'Update user profile',
    ]);

    // Default scopes when none specified
    Passport::defaultScopes([
        'read-posts',
        'read-profile',
    ]);
}
```

## Protecting Routes

```php
Route::middleware('auth:api')->group(function () {
    Route::get('/user', function (Request $request) {
        return $request->user();
    });
});

// With scope requirements
Route::get('/posts', [PostController::class, 'index'])
    ->middleware(['auth:api', 'scope:read-posts']);

Route::post('/posts', [PostController::class, 'store'])
    ->middleware(['auth:api', 'scopes:read-posts,write-posts']);
```

## Token Expiration

```php
// In AppServiceProvider
use Laravel\Passport\Passport;

public function boot(): void
{
    Passport::tokensExpireIn(now()->addDays(15));
    Passport::refreshTokensExpireIn(now()->addDays(30));
    Passport::personalAccessTokensExpireIn(now()->addMonths(6));
}
```

## Refreshing Tokens

```php
POST /oauth/token
{
    "grant_type": "refresh_token",
    "refresh_token": "REFRESH_TOKEN",
    "client_id": "CLIENT_ID",
    "client_secret": "CLIENT_SECRET",
    "scope": ""
}
```

## Resources

- [Laravel Passport](https://laravel.com/docs/12.x/passport) — Official Laravel Passport documentation

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*