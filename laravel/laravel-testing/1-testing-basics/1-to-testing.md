---
source_course: "laravel-testing"
source_lesson: "laravel-testing-introduction-to-testing"
---

# Introduction to Testing in Laravel

Testing ensures your application works as expected. Laravel is built with testing in mind and provides excellent tools for all types of tests.

## Why Test?

| Benefit | Description |
|---------|-------------|
| **Confidence** | Know your code works before deploying |
| **Refactoring Safety** | Change code without breaking features |
| **Documentation** | Tests show how code should behave |
| **Faster Development** | Catch bugs early, not in production |
| **Better Design** | Testing encourages better architecture |

## Types of Tests

```
┌─────────────────────────────────────────────────────────┐
│                      E2E Tests                          │
│              (Browser, full user flows)                 │
│                     Slowest, fewest                     │
├─────────────────────────────────────────────────────────┤
│                    Feature Tests                        │
│            (HTTP requests, database)                    │
│                   Medium speed & count                  │
├─────────────────────────────────────────────────────────┤
│                     Unit Tests                          │
│              (Single class/function)                    │
│                    Fastest, most                        │
└─────────────────────────────────────────────────────────┘
```

- **Unit Tests**: Test isolated pieces (a single class or method)
- **Feature Tests**: Test features (HTTP requests, multiple classes)
- **Browser Tests**: Test through a real browser (Laravel Dusk)

## Test Structure in Laravel

```
tests/
├── Feature/             # Feature tests
│   ├── ExampleTest.php
│   └── PostTest.php
├── Unit/                # Unit tests
│   ├── ExampleTest.php
│   └── UserTest.php
├── TestCase.php         # Base test class
└── CreatesApplication.php
```

## PHPUnit vs Pest

Laravel supports both testing frameworks:

### PHPUnit (Traditional)

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_the_application_returns_a_successful_response(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

### Pest (Modern, Fluent)

```php
<?php

test('the application returns a successful response', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

## Running Tests

```bash
# Run all tests
php artisan test

# With PHPUnit directly
./vendor/bin/phpunit

# Run specific test file
php artisan test tests/Feature/PostTest.php

# Run specific test method
php artisan test --filter test_user_can_view_posts

# Run tests in parallel
php artisan test --parallel

# Stop on first failure
php artisan test --stop-on-failure

# With coverage report
php artisan test --coverage
```

## Creating Tests

```bash
# Create a feature test
php artisan make:test PostTest

# Create a unit test
php artisan make:test UserTest --unit

# Create a Pest test
php artisan make:test PostTest --pest
```

## Test Environment

Tests use the `testing` environment:

```php
// phpunit.xml sets:
<env name="APP_ENV" value="testing"/>
<env name="DB_DATABASE" value=":memory:"/>  // SQLite in memory
```

You can also create `.env.testing` for test-specific configuration.

## Basic Assertions

```php
// PHPUnit assertions
$this->assertTrue($value);
$this->assertFalse($value);
$this->assertEquals($expected, $actual);
$this->assertNull($value);
$this->assertNotNull($value);
$this->assertCount(3, $array);
$this->assertContains('item', $array);
$this->assertInstanceOf(User::class, $user);

// Pest assertions
expect($value)->toBeTrue();
expect($value)->toBeFalse();
expect($actual)->toBe($expected);
expect($actual)->toEqual($expected);
expect($value)->toBeNull();
expect($array)->toHaveCount(3);
expect($array)->toContain('item');
expect($user)->toBeInstanceOf(User::class);
```

## Your First Test

```php
<?php

namespace Tests\Unit;

use App\Models\User;
use PHPUnit\Framework\TestCase;

class UserTest extends TestCase
{
    public function test_user_full_name(): void
    {
        $user = new User([
            'first_name' => 'John',
            'last_name' => 'Doe',
        ]);

        $this->assertEquals('John Doe', $user->fullName);
    }

    public function test_user_is_admin_by_default_false(): void
    {
        $user = new User();

        $this->assertFalse($user->is_admin);
    }
}
```

With Pest:

```php
<?php

use App\Models\User;

test('user full name', function () {
    $user = new User([
        'first_name' => 'John',
        'last_name' => 'Doe',
    ]);

    expect($user->fullName)->toBe('John Doe');
});

test('user is not admin by default', function () {
    $user = new User();

    expect($user->is_admin)->toBeFalse();
});
```

## Resources

- [Testing: Getting Started](https://laravel.com/docs/12.x/testing) — Official Laravel testing documentation

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*