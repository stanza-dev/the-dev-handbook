---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-email-verification"
---

# Email Verification

Email verification ensures users own the email addresses they register with. Laravel makes this easy to implement.

## Setting Up Email Verification

### 1. Implement the Contract

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;  // Add this
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail  // Add this
{
    use Notifiable;

    // ...
}
```

### 2. Database Migration

The users table needs `email_verified_at`:

```php
$table->timestamp('email_verified_at')->nullable();
```

### 3. Define Routes

```php
use Illuminate\Foundation\Auth\EmailVerificationRequest;

// Notice page
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');

// Verification handler
Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();  // Marks email as verified

    return redirect('/dashboard');
})->middleware(['auth', 'signed'])->name('verification.verify');

// Resend verification
Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Verification link sent!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

### 4. Protect Routes

```php
// Require email verification
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware(['auth', 'verified']);

// Or protect a group
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/billing', [BillingController::class, 'index']);
});
```

## The Verification Notice View

```blade
<!-- resources/views/auth/verify-email.blade.php -->

<x-layout>
    <div class="container">
        <h1>Verify Your Email Address</h1>

        @if (session('message'))
            <div class="alert alert-success">
                {{ session('message') }}
            </div>
        @endif

        <p>Before proceeding, please check your email for a verification link.</p>
        <p>If you did not receive the email:</p>

        <form method="POST" action="{{ route('verification.send') }}">
            @csrf
            <button type="submit">Resend Verification Email</button>
        </form>
    </div>
</x-layout>
```

## Customizing the Verification Email

```php
<?php

namespace App\Models;

use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements MustVerifyEmail
{
    /**
     * Send the email verification notification.
     */
    public function sendEmailVerificationNotification(): void
    {
        $this->notify(new VerifyEmail);

        // Or use your custom notification
        // $this->notify(new CustomVerifyEmail);
    }
}
```

### Custom Notification

```php
<?php

namespace App\Notifications;

use Illuminate\Auth\Notifications\VerifyEmail as BaseVerifyEmail;
use Illuminate\Notifications\Messages\MailMessage;

class CustomVerifyEmail extends BaseVerifyEmail
{
    public function toMail($notifiable): MailMessage
    {
        $verificationUrl = $this->verificationUrl($notifiable);

        return (new MailMessage)
            ->subject('Welcome! Please Verify Your Email')
            ->greeting('Hello ' . $notifiable->name . '!')
            ->line('Thanks for signing up. Please verify your email address.')
            ->action('Verify Email', $verificationUrl)
            ->line('This link will expire in 60 minutes.')
            ->line('If you did not create an account, no action is needed.');
    }
}
```

## Checking Verification Status

```php
// In a controller
if ($request->user()->hasVerifiedEmail()) {
    // Email is verified
}

// In Blade
@if (auth()->user()->hasVerifiedEmail())
    <span class="badge">✓ Verified</span>
@else
    <span class="badge">Pending verification</span>
@endif
```

## Events

Listen for verification events:

```php
// In EventServiceProvider
protected $listen = [
    Registered::class => [
        SendEmailVerificationNotification::class,
    ],
    Verified::class => [
        \App\Listeners\LogVerifiedUser::class,
    ],
];
```

```php
// app/Listeners/LogVerifiedUser.php
class LogVerifiedUser
{
    public function handle(Verified $event): void
    {
        Log::info('User verified email', [
            'user_id' => $event->user->id,
            'email' => $event->user->email,
        ]);
    }
}
```

## Manual Verification

```php
// Mark as verified manually
$user->markEmailAsVerified();

// Check verification
$user->hasVerifiedEmail();  // true/false

// Get verification timestamp
$user->email_verified_at;  // Carbon instance or null
```

## Resources

- [Email Verification](https://laravel.com/docs/12.x/verification) — Official email verification documentation

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*