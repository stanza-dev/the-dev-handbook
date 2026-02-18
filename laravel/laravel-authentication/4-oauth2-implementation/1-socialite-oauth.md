---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-socialite-oauth"
---

# Social Authentication with Socialite

Laravel Socialite provides a simple, fluent interface for OAuth authentication with Facebook, Google, Twitter, GitHub, and many other providers.

## Installing Socialite

```bash
composer require laravel/socialite
```

## Configuration

Add credentials in `config/services.php`:

```php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => env('GITHUB_REDIRECT_URI'),
],

'google' => [
    'client_id' => env('GOOGLE_CLIENT_ID'),
    'client_secret' => env('GOOGLE_CLIENT_SECRET'),
    'redirect' => env('GOOGLE_REDIRECT_URI'),
],
```

Add to `.env`:

```env
GITHUB_CLIENT_ID=your-client-id
GITHUB_CLIENT_SECRET=your-client-secret
GITHUB_REDIRECT_URI=http://localhost:8000/auth/github/callback

GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:8000/auth/google/callback
```

## OAuth Flow

```
1. User clicks "Login with GitHub"
   ┌──────────────────────────────────────────┐
   │ Your App → Redirect to GitHub           │
   └──────────────────────────────────────────┘
                    │
                    ▼
2. User authorizes your app on GitHub
   ┌──────────────────────────────────────────┐
   │ GitHub → Shows authorization screen     │
   │ User clicks "Authorize"                 │
   └──────────────────────────────────────────┘
                    │
                    ▼
3. GitHub redirects back with authorization code
   ┌──────────────────────────────────────────┐
   │ GitHub → Your callback URL              │
   │ ?code=abc123...                         │
   └──────────────────────────────────────────┘
                    │
                    ▼
4. Your app exchanges code for user data
   ┌──────────────────────────────────────────┐
   │ Socialite::driver('github')->user()     │
   │ Returns: name, email, avatar, etc.      │
   └──────────────────────────────────────────┘
                    │
                    ▼
5. Create or log in user
   ┌──────────────────────────────────────────┐
   │ Find/create user, log them in           │
   └──────────────────────────────────────────┘
```

## Defining Routes

```php
// routes/web.php
use App\Http\Controllers\Auth\SocialiteController;

Route::get('/auth/{provider}', [SocialiteController::class, 'redirect'])
    ->name('socialite.redirect');

Route::get('/auth/{provider}/callback', [SocialiteController::class, 'callback'])
    ->name('socialite.callback');
```

## The Controller

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Str;
use Laravel\Socialite\Facades\Socialite;

class SocialiteController extends Controller
{
    /**
     * Supported providers.
     */
    protected array $providers = ['github', 'google', 'facebook'];

    /**
     * Redirect to OAuth provider.
     */
    public function redirect(string $provider)
    {
        if (! in_array($provider, $this->providers)) {
            abort(404);
        }

        return Socialite::driver($provider)->redirect();
    }

    /**
     * Handle OAuth callback.
     */
    public function callback(string $provider)
    {
        if (! in_array($provider, $this->providers)) {
            abort(404);
        }

        try {
            $socialUser = Socialite::driver($provider)->user();
        } catch (\Exception $e) {
            return redirect()->route('login')
                ->with('error', 'Unable to authenticate with ' . ucfirst($provider));
        }

        // Find or create user
        $user = User::updateOrCreate(
            [
                'provider' => $provider,
                'provider_id' => $socialUser->getId(),
            ],
            [
                'name' => $socialUser->getName(),
                'email' => $socialUser->getEmail(),
                'avatar' => $socialUser->getAvatar(),
                'provider_token' => $socialUser->token,
                'provider_refresh_token' => $socialUser->refreshToken,
                'password' => bcrypt(Str::random(24)),  // Random password
            ]
        );

        Auth::login($user, remember: true);

        return redirect()->intended('/dashboard');
    }
}
```

## Database Migration

```php
// Add social auth columns to users table
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('provider')->nullable();
        $table->string('provider_id')->nullable();
        $table->string('provider_token')->nullable();
        $table->string('provider_refresh_token')->nullable();
        $table->string('avatar')->nullable();
        $table->string('password')->nullable()->change();  // Make nullable for social users

        $table->unique(['provider', 'provider_id']);
    });
}
```

## The Socialite User Object

```php
$socialUser = Socialite::driver('github')->user();

$socialUser->getId();           // Provider's user ID
$socialUser->getNickname();     // Username (if available)
$socialUser->getName();         // Full name
$socialUser->getEmail();        // Email address
$socialUser->getAvatar();       // Avatar URL

$socialUser->token;             // Access token
$socialUser->refreshToken;      // Refresh token (if available)
$socialUser->expiresIn;         // Token expiration (seconds)

// Raw data from provider
$socialUser->user;              // Full response array
$socialUser->user['login'];     // Access specific fields
```

## Handling Existing Users

```php
public function callback(string $provider)
{
    $socialUser = Socialite::driver($provider)->user();

    // Check if user exists with this email
    $existingUser = User::where('email', $socialUser->getEmail())->first();

    if ($existingUser) {
        // Link social account to existing user
        if (! $existingUser->provider) {
            $existingUser->update([
                'provider' => $provider,
                'provider_id' => $socialUser->getId(),
            ]);
        }

        Auth::login($existingUser);
    } else {
        // Create new user
        $user = User::create([
            'name' => $socialUser->getName(),
            'email' => $socialUser->getEmail(),
            'provider' => $provider,
            'provider_id' => $socialUser->getId(),
        ]);

        Auth::login($user);
    }

    return redirect()->intended('/dashboard');
}
```

## View Integration

```blade
<div class="social-login">
    <a href="{{ route('socialite.redirect', 'github') }}" 
       class="btn btn-github">
        <svg>...</svg> Login with GitHub
    </a>
    
    <a href="{{ route('socialite.redirect', 'google') }}" 
       class="btn btn-google">
        <svg>...</svg> Login with Google
    </a>
</div>
```

## Additional Scopes

```php
// Request additional permissions
return Socialite::driver('github')
    ->scopes(['read:user', 'user:email', 'repo'])
    ->redirect();

// Stateless for API usage
$user = Socialite::driver('github')->stateless()->user();
```

## Resources

- [Laravel Socialite](https://laravel.com/docs/12.x/socialite) — Official Laravel Socialite documentation

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*