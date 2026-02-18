---
source_course: "laravel-testing"
source_lesson: "laravel-testing-dusk-setup"
---

# Getting Started with Laravel Dusk

Laravel Dusk provides expressive, browser-based testing for your applications. It tests through a real browser, simulating actual user interactions.

## What is Browser Testing?

Browser tests (E2E tests):
- Run in a real browser (Chrome)
- Test JavaScript interactions
- Verify the actual user experience
- Are slower but catch issues other tests miss

```
┌─────────────────────────────────────────────────────────┐
│                    Test Pyramid                         │
│                                                         │
│                        /\                               │
│                       /  \                              │
│                      / E2E\   <- Dusk (Few)            │
│                     /──────\                            │
│                    /Feature \  <- HTTP Tests (Some)    │
│                   /──────────\                          │
│                  /    Unit    \ <- Unit Tests (Many)   │
│                 ────────────────                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Installation

```bash
composer require laravel/dusk --dev
php artisan dusk:install
```

This creates:
- `tests/Browser/` - Browser test directory
- `tests/Browser/ExampleTest.php` - Example test
- `tests/DuskTestCase.php` - Base test class

## Basic Dusk Test

```php
<?php

namespace Tests\Browser;

use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    public function test_basic_example(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertSee('Laravel');
        });
    }
}
```

## Running Dusk Tests

```bash
# Run all Dusk tests
php artisan dusk

# Run specific test file
php artisan dusk tests/Browser/LoginTest.php

# Run specific test method
php artisan dusk --filter test_user_can_login

# Run in headed mode (see the browser)
php artisan dusk --browse
```

## Environment Setup

Create `.env.dusk.local` for Dusk-specific configuration:

```env
APP_ENV=testing
APP_URL=http://localhost:8000
DB_CONNECTION=mysql
DB_DATABASE=dusk_testing
```

## Navigation and URLs

```php
$this->browse(function (Browser $browser) {
    // Visit URL
    $browser->visit('/posts');
    $browser->visit(route('posts.index'));

    // Navigation
    $browser->back();
    $browser->forward();
    $browser->refresh();

    // Links
    $browser->clickLink('View Post');

    // Assert current URL
    $browser->assertPathIs('/posts');
    $browser->assertPathIsNot('/login');
    $browser->assertUrlIs('http://localhost:8000/posts');
    $browser->assertRouteIs('posts.index');
    $browser->assertQueryStringHas('page');
    $browser->assertQueryStringHas('page', '2');
});
```

## Interacting with Elements

### Clicking

```php
$browser->click('.submit-button');
$browser->click('@submit-button');  // Dusk selector
$browser->clickLink('Log In');
$browser->clickAtXPath('//button[@id="submit"]');
```

### Typing

```php
$browser->type('email', 'user@example.com');
$browser->type('@email-input', 'user@example.com');
$browser->typeSlowly('email', 'user@example.com'); // With delay
$browser->clear('email');
$browser->append('email', '.com');
```

### Forms

```php
$browser->select('country', 'us');
$browser->select('country');  // Random selection

$browser->check('terms');
$browser->uncheck('newsletter');

$browser->radio('plan', 'premium');

$browser->attach('resume', __DIR__ . '/files/resume.pdf');

$browser->press('Submit');
$browser->press('@submit-button');
```

## Dusk Selectors

Add `dusk` attributes to HTML for stable selectors:

```html
<!-- In your Blade template -->
<button dusk="submit-button">Submit</button>
<input dusk="email-input" type="email">
```

```php
// In your test
$browser->click('@submit-button');
$browser->type('@email-input', 'test@example.com');
```

## Waiting for Elements

```php
// Wait for element
$browser->waitFor('.modal');
$browser->waitFor('@modal', 10);  // Wait up to 10 seconds

// Wait for text
$browser->waitForText('Success');

// Wait for element to disappear
$browser->waitUntilMissing('.loading');

// Wait for location
$browser->waitForLocation('/dashboard');

// Wait for reload
$browser->waitForReload();

// Generic wait
$browser->pause(1000);  // 1 second

// Wait with callback
$browser->waitUsing(10, 100, function () use ($browser) {
    return $browser->element('.ready') !== null;
});
```

## Complete Login Test Example

```php
<?php

namespace Tests\Browser;

use App\Models\User;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class LoginTest extends DuskTestCase
{
    public function test_user_can_login(): void
    {
        $user = User::factory()->create([
            'email' => 'test@example.com',
            'password' => bcrypt('password'),
        ]);

        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                    ->type('email', 'test@example.com')
                    ->type('password', 'password')
                    ->press('Log in')
                    ->assertPathIs('/dashboard')
                    ->assertSee('Dashboard');
        });
    }

    public function test_login_fails_with_wrong_password(): void
    {
        $user = User::factory()->create();

        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                    ->type('email', $user->email)
                    ->type('password', 'wrong-password')
                    ->press('Log in')
                    ->assertPathIs('/login')
                    ->assertSee('credentials do not match');
        });
    }

    public function test_user_can_logout(): void
    {
        $user = User::factory()->create();

        $this->browse(function (Browser $browser) use ($user) {
            $browser->loginAs($user)
                    ->visit('/dashboard')
                    ->clickLink('Log Out')
                    ->assertPathIs('/');
        });
    }
}
```

## Resources

- [Laravel Dusk](https://laravel.com/docs/12.x/dusk) — Official Laravel Dusk documentation

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*