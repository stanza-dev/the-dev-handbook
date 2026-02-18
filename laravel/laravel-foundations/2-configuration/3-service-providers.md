---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-service-providers"
---

# Service Providers and Bootstrapping

Service providers are the central place for configuring and bootstrapping your Laravel application. Every major Laravel feature is bootstrapped via a service provider.

## What Are Service Providers?

Service providers are classes responsible for:

1. **Registering** things into the service container
2. **Booting** the application with necessary setup

```
Laravel Application Boot Process
┌─────────────────────────────────────────┐
│  1. Load Configuration                  │
│  2. Register Service Providers          │
│     - register() method called          │
│  3. Boot Service Providers              │
│     - boot() method called              │
│  4. Handle Request                      │
└─────────────────────────────────────────┘
```

## Creating a Service Provider

Generate a new provider:

```bash
php artisan make:provider RiakServiceProvider
```

This creates `app/Providers/RiakServiceProvider.php`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // Bind services to the container
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // Run after all providers are registered
    }
}
```

## The register() Method

Use `register()` to bind things into the service container:

```php
public function register(): void
{
    // Bind a class
    $this->app->bind(PaymentGateway::class, function ($app) {
        return new StripeGateway(config('services.stripe.key'));
    });

    // Bind a singleton (one instance shared)
    $this->app->singleton(ReportGenerator::class, function ($app) {
        return new ReportGenerator();
    });
}
```

**Rule**: Only bind things in `register()`. Don't use other services here—they might not be registered yet.

## The boot() Method

Use `boot()` after all providers are registered:

```php
public function boot(): void
{
    // Configure view composers
    View::composer('dashboard', function ($view) {
        $view->with('notifications', auth()->user()->notifications);
    });

    // Register custom validation rules
    Validator::extend('phone', function ($attribute, $value) {
        return preg_match('/^[0-9]{10}$/', $value);
    });

    // Publish package assets
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php'),
    ]);
}
```

## Built-in Service Providers

Laravel includes many providers in `bootstrap/providers.php`:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

Laravel 12 uses automatic package discovery, so most providers are loaded automatically.

## The AppServiceProvider

Your main application provider at `app/Providers/AppServiceProvider.php`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\URL;
use Illuminate\Pagination\Paginator;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Register application bindings
    }

    public function boot(): void
    {
        // Force HTTPS in production
        if ($this->app->environment('production')) {
            URL::forceScheme('https');
        }

        // Use Bootstrap for pagination
        Paginator::useBootstrap();

        // Custom model binding
        Model::preventLazyLoading(! $this->app->isProduction());
    }
}
```

## Registering Your Provider

Add custom providers to `bootstrap/providers.php`:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\RiakServiceProvider::class,  // Add your provider
];
```

## Deferred Providers

For performance, defer loading until needed:

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    public function register(): void
    {
        $this->app->singleton(Connection::class, function () {
            return new Connection(config('riak'));
        });
    }

    /**
     * Get the services provided by the provider.
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```

## The Service Container

Service providers work with Laravel's service container. The container manages class dependencies and performs dependency injection.

```php
// When you type-hint a class, Laravel resolves it from the container
class UserController extends Controller
{
    public function __construct(
        private PaymentGateway $gateway  // Auto-injected!
    ) {}

    public function charge()
    {
        $this->gateway->charge(100);
    }
}
```

## Resources

- [Service Providers](https://laravel.com/docs/12.x/providers) — Official documentation on service providers
- [Service Container](https://laravel.com/docs/12.x/container) — Understanding Laravel's dependency injection container

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*