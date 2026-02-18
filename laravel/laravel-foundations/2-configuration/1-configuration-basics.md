---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-configuration-basics"
---

# Understanding Laravel Configuration

Laravel's configuration system is both powerful and flexible. All configuration files are stored in the `config` directory, and each option is well-documented.

## The config Directory

Laravel includes configuration files for every major feature:

```
config/
├── app.php         # Application settings (name, timezone, locale)
├── auth.php        # Authentication guards and providers
├── cache.php       # Cache drivers and settings
├── database.php    # Database connections
├── filesystems.php # File storage disks
├── mail.php        # Email settings
├── queue.php       # Queue connections
├── services.php    # Third-party service credentials
└── session.php     # Session driver and settings
```

## Accessing Configuration Values

Use the `config()` helper to access configuration values:

```php
// Get a configuration value
$appName = config('app.name');         // 'Laravel'
$timezone = config('app.timezone');    // 'UTC'

// Nested configuration
$driver = config('database.default');  // 'sqlite'

// With a default value
$value = config('app.custom_key', 'default');
```

### Dot Notation

Configuration uses dot notation where:
- First segment = filename (without .php)
- Following segments = array keys

```php
// config/app.php
return [
    'name' => env('APP_NAME', 'Laravel'),
    'env' => env('APP_ENV', 'production'),
    'debug' => env('APP_DEBUG', false),
];

// Accessing these values
config('app.name');   // First 'app' = file, 'name' = key
config('app.debug');
```

## Setting Configuration at Runtime

You can set configuration values at runtime:

```php
// Set a single value
config(['app.timezone' => 'America/New_York']);

// Set multiple values
config([
    'app.timezone' => 'America/New_York',
    'app.debug' => true,
]);
```

**Note**: Runtime configuration changes only last for the current request.

## Configuration Structure

Each config file returns a PHP array:

```php
// config/app.php
<?php

return [
    /*
    |--------------------------------------------------------------------------
    | Application Name
    |--------------------------------------------------------------------------
    |
    | This value is the name of your application, which will be used when the
    | framework needs to place the application's name in a notification or
    | other UI elements where an application name needs to be displayed.
    |
    */

    'name' => env('APP_NAME', 'Laravel'),

    'env' => env('APP_ENV', 'production'),

    'debug' => (bool) env('APP_DEBUG', false),

    'url' => env('APP_URL', 'http://localhost'),

    'timezone' => env('APP_TIMEZONE', 'UTC'),

    'locale' => env('APP_LOCALE', 'en'),
];
```

## The config/app.php File

The main application configuration includes:

```php
return [
    // Application name
    'name' => env('APP_NAME', 'Laravel'),

    // Environment (local, staging, production)
    'env' => env('APP_ENV', 'production'),

    // Debug mode (show detailed errors)
    'debug' => (bool) env('APP_DEBUG', false),

    // Application URL
    'url' => env('APP_URL', 'http://localhost'),

    // Timezone for PHP date functions
    'timezone' => env('APP_TIMEZONE', 'UTC'),

    // Locale for translations
    'locale' => env('APP_LOCALE', 'en'),

    // Fallback locale
    'fallback_locale' => env('APP_FALLBACK_LOCALE', 'en'),

    // Application key (for encryption)
    'key' => env('APP_KEY'),
    'cipher' => 'AES-256-CBC',
];
```

## Debug Mode

**Debug mode** shows detailed error messages with stack traces:

```php
// .env
APP_DEBUG=true   // Development: shows full errors
APP_DEBUG=false  // Production: shows generic error page
```

**Warning**: Never enable debug mode in production! It exposes sensitive information.

## Maintenance Mode

Laravel can put your application in maintenance mode:

```bash
# Enable maintenance mode
php artisan down

# With a custom message
php artisan down --message="Upgrading Database"

# Allow certain IPs
php artisan down --allow=127.0.0.1

# Disable maintenance mode
php artisan up
```

## Resources

- [Configuration](https://laravel.com/docs/12.x/configuration) — Official Laravel configuration documentation

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*