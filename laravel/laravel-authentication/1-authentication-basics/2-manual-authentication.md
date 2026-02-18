---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-manual-authentication"
---

# Manual Authentication Implementation

While starter kits handle authentication for you, understanding manual implementation helps you customize and debug authentication.

## Login Implementation

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Show the login form.
     */
    public function showLoginForm()
    {
        return view('auth.login');
    }

    /**
     * Handle a login request.
     */
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        // Attempt to authenticate
        if (Auth::attempt($credentials, $request->boolean('remember'))) {
            // Regenerate session for security
            $request->session()->regenerate();

            return redirect()->intended('dashboard');
        }

        // Authentication failed
        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ])->onlyInput('email');
    }
}
```

### The attempt() Method

```php
// Basic attempt
Auth::attempt(['email' => $email, 'password' => $password]);

// With "remember me"
Auth::attempt($credentials, $remember = true);

// Additional conditions
Auth::attempt([
    'email' => $email,
    'password' => $password,
    'active' => true,  // User must be active
]);

// Using specific guard
Auth::guard('admin')->attempt($credentials);
```

## Logout Implementation

```php
public function logout(Request $request)
{
    Auth::logout();

    // Invalidate session
    $request->session()->invalidate();

    // Regenerate CSRF token
    $request->session()->regenerateToken();

    return redirect('/');
}
```

## Registration

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Models\User;
use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;

class RegisterController extends Controller
{
    public function showRegistrationForm()
    {
        return view('auth.register');
    }

    public function register(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed',
        ]);

        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        // Log the user in
        Auth::login($user);

        return redirect('dashboard');
    }
}
```

## Password Reset

Laravel provides built-in password reset:

```php
// routes/web.php
use Illuminate\Support\Facades\Password;

// Show request form
Route::get('/forgot-password', function () {
    return view('auth.forgot-password');
})->middleware('guest')->name('password.request');

// Send reset link
Route::post('/forgot-password', function (Request $request) {
    $request->validate(['email' => 'required|email']);

    $status = Password::sendResetLink(
        $request->only('email')
    );

    return $status === Password::RESET_LINK_SENT
        ? back()->with(['status' => __($status)])
        : back()->withErrors(['email' => __($status)]);
})->middleware('guest')->name('password.email');

// Show reset form
Route::get('/reset-password/{token}', function (string $token) {
    return view('auth.reset-password', ['token' => $token]);
})->middleware('guest')->name('password.reset');

// Handle reset
Route::post('/reset-password', function (Request $request) {
    $request->validate([
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|min:8|confirmed',
    ]);

    $status = Password::reset(
        $request->only('email', 'password', 'password_confirmation', 'token'),
        function (User $user, string $password) {
            $user->forceFill([
                'password' => Hash::make($password)
            ])->setRememberToken(Str::random(60));

            $user->save();
        }
    );

    return $status === Password::PASSWORD_RESET
        ? redirect()->route('login')->with('status', __($status))
        : back()->withErrors(['email' => [__($status)]]);
})->middleware('guest')->name('password.update');
```

## Route Protection

```php
// Protect individual routes
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('auth');

// Protect route groups
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/profile', [ProfileController::class, 'show']);
});

// Guest-only routes (redirect if authenticated)
Route::get('/login', [LoginController::class, 'showLoginForm'])
    ->middleware('guest');
```

## Intended Redirect

Remember where users wanted to go before login:

```php
// After successful login
return redirect()->intended('dashboard');

// 'intended' checks:
// 1. Was user redirected to login? Go back there.
// 2. Otherwise, go to 'dashboard' (fallback)
```

## Login Throttling

Prevent brute force attacks:

```php
use Illuminate\Support\Facades\RateLimiter;

public function login(Request $request)
{
    $throttleKey = strtolower($request->email).'|'.$request->ip();

    if (RateLimiter::tooManyAttempts($throttleKey, 5)) {
        $seconds = RateLimiter::availableIn($throttleKey);
        return back()->withErrors([
            'email' => "Too many attempts. Try again in {$seconds} seconds.",
        ]);
    }

    if (Auth::attempt($request->only('email', 'password'))) {
        RateLimiter::clear($throttleKey);
        return redirect()->intended('dashboard');
    }

    RateLimiter::hit($throttleKey);
    return back()->withErrors(['email' => 'Invalid credentials.']);
}
```

## Resources

- [Manually Authenticating Users](https://laravel.com/docs/12.x/authentication#authenticating-users) — Manual authentication implementation guide
- [Password Reset](https://laravel.com/docs/12.x/passwords) — Password reset documentation

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*