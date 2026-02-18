---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-security-best-practices"
---

# Security Best Practices

Laravel provides many security features out of the box, but proper configuration and awareness are essential for building secure applications.

## Password Security

### Strong Password Requirements

```php
use Illuminate\Validation\Rules\Password;

$request->validate([
    'password' => [
        'required',
        'confirmed',
        Password::min(8)
            ->letters()      // At least one letter
            ->mixedCase()    // Upper and lowercase
            ->numbers()      // At least one number
            ->symbols()      // At least one symbol
            ->uncompromised(), // Not in data breaches
    ],
]);
```

### Default Password Rules

```php
// In AppServiceProvider
use Illuminate\Validation\Rules\Password;

public function boot(): void
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
            ? $rule->mixedCase()->numbers()->symbols()->uncompromised()
            : $rule;
    });
}

// Usage
$request->validate([
    'password' => ['required', 'confirmed', Password::defaults()],
]);
```

## Rate Limiting

### Login Throttling

```php
use Illuminate\Support\Facades\RateLimiter;

public function login(Request $request)
{
    $throttleKey = Str::lower($request->email) . '|' . $request->ip();

    if (RateLimiter::tooManyAttempts($throttleKey, 5)) {
        $seconds = RateLimiter::availableIn($throttleKey);

        throw ValidationException::withMessages([
            'email' => ["Too many attempts. Try again in {$seconds} seconds."],
        ]);
    }

    if (! Auth::attempt($request->only('email', 'password'))) {
        RateLimiter::hit($throttleKey, 60); // Lock for 60 seconds

        throw ValidationException::withMessages([
            'email' => ['Invalid credentials.'],
        ]);
    }

    RateLimiter::clear($throttleKey);

    return redirect()->intended('/dashboard');
}
```

### API Rate Limiting

```php
// In AppServiceProvider
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

RateLimiter::for('sensitive', function (Request $request) {
    return Limit::perMinute(10)->by($request->user()->id);
});
```

## Session Security

```php
// config/session.php

'lifetime' => 120,        // Session lifetime in minutes
'expire_on_close' => false,
'encrypt' => true,        // Encrypt session data
'http_only' => true,      // JavaScript can't access cookies
'secure' => env('SESSION_SECURE_COOKIE', true), // HTTPS only
'same_site' => 'lax',     // CSRF protection
```

### Session Regeneration

```php
// Always regenerate session after login
public function login(Request $request)
{
    if (Auth::attempt($request->only('email', 'password'))) {
        $request->session()->regenerate();
        return redirect()->intended('/dashboard');
    }
}

// Invalidate session on logout
public function logout(Request $request)
{
    Auth::logout();
    $request->session()->invalidate();
    $request->session()->regenerateToken();
    return redirect('/');
}
```

## CSRF Protection

```php
// All POST, PUT, PATCH, DELETE forms need @csrf
<form method="POST" action="/profile">
    @csrf
    <!-- form fields -->
</form>

// For AJAX requests, include the token in headers
<meta name="csrf-token" content="{{ csrf_token() }}">

<script>
axios.defaults.headers.common['X-CSRF-TOKEN'] = 
    document.querySelector('meta[name="csrf-token"]').content;
</script>
```

## SQL Injection Prevention

```php
// ✅ Safe - Eloquent automatically escapes
User::where('email', $request->email)->first();

// ✅ Safe - Query builder with bindings
DB::table('users')->where('email', $request->email)->first();

// ❌ DANGEROUS - Never interpolate user input
DB::select("SELECT * FROM users WHERE email = '{$request->email}'");

// ✅ Safe - Use bindings for raw queries
DB::select('SELECT * FROM users WHERE email = ?', [$request->email]);
```

## XSS Prevention

```blade
{{-- ✅ Safe - Escaped by default --}}
<h1>{{ $user->name }}</h1>

{{-- ❌ Dangerous - Unescaped output --}}
{!! $user->bio !!}  {{-- Only if you trust the content! --}}

{{-- ✅ Safe for HTML content --}}
{!! clean($user->bio) !!}  {{-- Use a sanitizer like HTMLPurifier --}}
```

## Force HTTPS

```php
// In AppServiceProvider
public function boot(): void
{
    if ($this->app->environment('production')) {
        URL::forceScheme('https');
    }
}
```

## Secure Headers Middleware

```php
class SecurityHeaders
{
    public function handle(Request $request, Closure $next)
    {
        $response = $next($request);

        $response->headers->set('X-Frame-Options', 'SAMEORIGIN');
        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-XSS-Protection', '1; mode=block');
        $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
        $response->headers->set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');

        if (app()->environment('production')) {
            $response->headers->set(
                'Strict-Transport-Security',
                'max-age=31536000; includeSubDomains'
            );
        }

        return $response;
    }
}
```

## Environment Security

```env
# Never expose in production
APP_DEBUG=false
APP_ENV=production

# Use strong keys
APP_KEY=base64:generated-key-here
```

```php
// Never expose sensitive config
// config/app.php
'debug' => (bool) env('APP_DEBUG', false),
```

## Resources

- [Laravel Security](https://laravel.com/docs/12.x/authentication) — Laravel authentication and security documentation

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*