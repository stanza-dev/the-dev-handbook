---
source_course: "laravel-testing"
source_lesson: "laravel-testing-dusk-advanced"
---

# Advanced Dusk Techniques

Master advanced browser testing patterns for complex interactions and JavaScript-heavy applications.

## Page Objects

Page Objects encapsulate page-specific logic:

```bash
php artisan dusk:page LoginPage
```

```php
// tests/Browser/Pages/LoginPage.php
<?php

namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Page;

class LoginPage extends Page
{
    public function url(): string
    {
        return '/login';
    }

    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url())
                ->assertSee('Log in');
    }

    public function elements(): array
    {
        return [
            '@email' => 'input[name="email"]',
            '@password' => 'input[name="password"]',
            '@submit' => 'button[type="submit"]',
            '@error' => '.text-red-600',
        ];
    }

    public function login(Browser $browser, string $email, string $password): void
    {
        $browser->type('@email', $email)
                ->type('@password', $password)
                ->click('@submit');
    }

    public function assertHasError(Browser $browser, string $message): void
    {
        $browser->assertSeeIn('@error', $message);
    }
}
```

Using the Page Object:

```php
public function test_user_can_login(): void
{
    $user = User::factory()->create(['password' => bcrypt('password')]);

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit(new LoginPage)
                ->login($user->email, 'password')
                ->assertPathIs('/dashboard');
    });
}

public function test_shows_error_for_invalid_credentials(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit(new LoginPage)
                ->login('wrong@example.com', 'wrongpassword')
                ->on(new LoginPage)  // Still on login page
                ->assertHasError('credentials do not match');
    });
}
```

## Components

Reusable components for repeated UI elements:

```bash
php artisan dusk:component DropdownComponent
```

```php
// tests/Browser/Components/DropdownComponent.php
<?php

namespace Tests\Browser\Components;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Component as BaseComponent;

class DropdownComponent extends BaseComponent
{
    public function selector(): string
    {
        return '.dropdown';
    }

    public function assert(Browser $browser): void
    {
        $browser->assertVisible($this->selector());
    }

    public function elements(): array
    {
        return [
            '@trigger' => '.dropdown-trigger',
            '@menu' => '.dropdown-menu',
            '@items' => '.dropdown-item',
        ];
    }

    public function open(Browser $browser): void
    {
        $browser->click('@trigger')
                ->waitFor('@menu');
    }

    public function selectItem(Browser $browser, string $text): void
    {
        $this->open($browser);
        $browser->clickLink($text);
    }
}
```

```php
public function test_user_can_change_settings(): void
{
    $this->browse(function (Browser $browser) {
        $browser->loginAs($this->user)
                ->visit('/dashboard')
                ->within(new DropdownComponent, function (Browser $browser) {
                    $browser->selectItem('Settings');
                })
                ->assertPathIs('/settings');
    });
}
```

## Testing JavaScript Interactions

```php
public function test_modal_opens_and_closes(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/posts')
                ->click('@create-post-button')
                ->waitFor('@modal')
                ->assertVisible('@modal')
                ->type('@modal-title', 'My Post')
                ->within('@modal', function (Browser $modal) {
                    $modal->press('Cancel');
                })
                ->waitUntilMissing('@modal')
                ->assertMissing('@modal');
    });
}

public function test_infinite_scroll(): void
{
    Post::factory()->count(50)->create();

    $this->browse(function (Browser $browser) {
        $browser->visit('/posts')
                ->assertSeeIn('.post-count', '20')  // Initial load
                ->scrollToBottom()
                ->waitForText('Loading more...')
                ->waitUntilMissingText('Loading more...')
                ->assertSeeIn('.post-count', '40');  // After scroll
    });
}
```

## Handling JavaScript Dialogs

```php
public function test_delete_confirmation(): void
{
    $post = Post::factory()->create();

    $this->browse(function (Browser $browser) use ($post) {
        // Accept the dialog
        $browser->loginAs($post->user)
                ->visit('/posts/' . $post->id)
                ->click('@delete-button')
                ->acceptDialog()
                ->assertPathIs('/posts')
                ->assertDontSee($post->title);
    });
}

public function test_cancel_delete(): void
{
    $post = Post::factory()->create();

    $this->browse(function (Browser $browser) use ($post) {
        $browser->loginAs($post->user)
                ->visit('/posts/' . $post->id)
                ->click('@delete-button')
                ->dismissDialog()
                ->assertPathIs('/posts/' . $post->id);
    });
}

public function test_prompt_dialog(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/settings')
                ->click('@rename-button')
                ->typeInDialog('New Name')
                ->acceptDialog()
                ->assertSee('New Name');
    });
}
```

## Screenshots and Console

```php
public function test_with_debugging(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/complex-page')
                ->screenshot('step-1-initial')
                ->click('@action')
                ->screenshot('step-2-after-click')
                ->assertSee('Result');
    });
}

// Screenshots saved to tests/Browser/screenshots/

// Automatic screenshots on failure (in DuskTestCase)
protected function captureFailuresFor()
{
    return collect($this->storeScreenshotsAt());
}
```

## Console Logs

```php
public function test_no_javascript_errors(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
                ->assertNoJavascriptErrors();
    });
}

public function test_check_console_output(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/');

        $logs = $browser->driver->manage()->getLog('browser');

        foreach ($logs as $log) {
            $this->assertNotEquals('SEVERE', $log['level']);
        }
    });
}
```

## Multiple Browsers

Test interactions between users:

```php
public function test_realtime_chat(): void
{
    $alice = User::factory()->create(['name' => 'Alice']);
    $bob = User::factory()->create(['name' => 'Bob']);

    $this->browse(function (Browser $first, Browser $second) use ($alice, $bob) {
        // Alice joins chat
        $first->loginAs($alice)
              ->visit('/chat')
              ->waitFor('@chat-ready');

        // Bob joins chat
        $second->loginAs($bob)
               ->visit('/chat')
               ->waitFor('@chat-ready');

        // Alice sends message
        $first->type('@message-input', 'Hello Bob!')
              ->press('Send');

        // Bob receives message
        $second->waitForText('Hello Bob!')
               ->assertSee('Alice: Hello Bob!');

        // Bob replies
        $second->type('@message-input', 'Hi Alice!')
               ->press('Send');

        // Alice receives reply
        $first->waitForText('Hi Alice!')
              ->assertSee('Bob: Hi Alice!');
    });
}
```

## Resources

- [Dusk Pages](https://laravel.com/docs/12.x/dusk#pages) — Page Objects in Laravel Dusk

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*