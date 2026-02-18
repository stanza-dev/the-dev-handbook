---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-two-factor-auth"
---

# Two-Factor Authentication

Two-factor authentication (2FA) adds an extra layer of security by requiring users to provide a second verification method beyond their password.

## How 2FA Works

```
┌─────────────────────────────────────────────────────────────────┐
│                 TWO-FACTOR AUTHENTICATION                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Login with Email + Password                            │
│          ┌──────────────────────────────────────┐               │
│          │ Something you KNOW                    │               │
│          └──────────────────────────────────────┘               │
│                         │                                        │
│                         ▼                                        │
│  Step 2: Verify with TOTP Code or Recovery Code                 │
│          ┌──────────────────────────────────────┐               │
│          │ Something you HAVE (phone/app)        │               │
│          └──────────────────────────────────────┘               │
│                         │                                        │
│                         ▼                                        │
│  Access Granted                                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Using Laravel Fortify

Laravel Fortify provides 2FA out of the box:

```bash
composer require laravel/fortify

php artisan fortify:install

php artisan migrate
```

Enable 2FA in `config/fortify.php`:

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::updateProfileInformation(),
    Features::updatePasswords(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

## Manual Implementation

### Installation

```bash
composer require pragmarx/google2fa-laravel
composer require bacon/bacon-qr-code

php artisan vendor:publish --provider="PragmaRX\Google2FALaravel\ServiceProvider"
```

### Migration

```php
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->text('two_factor_secret')->nullable();
        $table->text('two_factor_recovery_codes')->nullable();
        $table->timestamp('two_factor_confirmed_at')->nullable();
    });
}
```

### User Model

```php
use PragmaRX\Google2FA\Google2FA;
use Illuminate\Support\Str;

class User extends Authenticatable
{
    protected $hidden = [
        'password',
        'remember_token',
        'two_factor_secret',
        'two_factor_recovery_codes',
    ];

    protected function casts(): array
    {
        return [
            'two_factor_secret' => 'encrypted',
            'two_factor_recovery_codes' => 'encrypted:array',
        ];
    }

    public function hasTwoFactorEnabled(): bool
    {
        return ! is_null($this->two_factor_confirmed_at);
    }

    public function enableTwoFactor(): void
    {
        $google2fa = new Google2FA();

        $this->two_factor_secret = $google2fa->generateSecretKey();
        $this->two_factor_recovery_codes = $this->generateRecoveryCodes();
        $this->save();
    }

    public function confirmTwoFactor(string $code): bool
    {
        $google2fa = new Google2FA();

        if ($google2fa->verifyKey($this->two_factor_secret, $code)) {
            $this->two_factor_confirmed_at = now();
            $this->save();
            return true;
        }

        return false;
    }

    public function disableTwoFactor(): void
    {
        $this->two_factor_secret = null;
        $this->two_factor_recovery_codes = null;
        $this->two_factor_confirmed_at = null;
        $this->save();
    }

    protected function generateRecoveryCodes(): array
    {
        return collect(range(1, 8))
            ->map(fn () => Str::random(10) . '-' . Str::random(10))
            ->all();
    }

    public function getQrCodeUrl(): string
    {
        $google2fa = new Google2FA();

        return $google2fa->getQRCodeUrl(
            config('app.name'),
            $this->email,
            $this->two_factor_secret
        );
    }
}
```

### Controller

```php
class TwoFactorController extends Controller
{
    public function show(Request $request)
    {
        $user = $request->user();

        if ($user->hasTwoFactorEnabled()) {
            return view('settings.two-factor.enabled');
        }

        // Generate secret if not exists
        if (! $user->two_factor_secret) {
            $user->enableTwoFactor();
        }

        // Generate QR code
        $qrCode = (new \BaconQrCode\Renderer\Image\SvgImageBackEnd())
            ->render($user->getQrCodeUrl());

        return view('settings.two-factor.setup', [
            'qrCode' => $qrCode,
            'secret' => $user->two_factor_secret,
            'recoveryCodes' => $user->two_factor_recovery_codes,
        ]);
    }

    public function confirm(Request $request)
    {
        $request->validate([
            'code' => 'required|string|size:6',
        ]);

        if ($request->user()->confirmTwoFactor($request->code)) {
            return redirect()->route('settings')
                ->with('success', '2FA enabled successfully!');
        }

        return back()->withErrors(['code' => 'Invalid verification code.']);
    }

    public function disable(Request $request)
    {
        $request->validate([
            'password' => 'required|current_password',
        ]);

        $request->user()->disableTwoFactor();

        return redirect()->route('settings')
            ->with('success', '2FA disabled.');
    }
}
```

### Login with 2FA

```php
class LoginController extends Controller
{
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        if (! Auth::attempt($credentials)) {
            return back()->withErrors(['email' => 'Invalid credentials.']);
        }

        $user = Auth::user();

        if ($user->hasTwoFactorEnabled()) {
            Auth::logout();

            $request->session()->put('login.id', $user->id);

            return redirect()->route('two-factor.challenge');
        }

        return redirect()->intended('/dashboard');
    }
}
```

## Resources

- [Laravel Fortify](https://laravel.com/docs/12.x/fortify#two-factor-authentication) — Official documentation on two-factor authentication

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*