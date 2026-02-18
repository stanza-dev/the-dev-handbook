---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-environment-variables"
---

# Environment Variables and .env Files

Laravel uses environment variables to manage settings that differ between development, staging, and production. This keeps sensitive information out of your codebase.

## Why Environment Variables?

```
┌─────────────────────────────────────────────────────────────┐
│                     Your Codebase                           │
│                    (version control)                        │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  config/database.php                                   │  │
│  │  'host' => env('DB_HOST', 'localhost')                │  │
│  │  'database' => env('DB_DATABASE', 'forge')            │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                      │
         Uses values from environment
                      │
    ┌─────────────────┴─────────────────┐
    ▼                                   ▼
┌──────────────┐                 ┌──────────────┐
│  .env (local) │                 │.env (prod)   │
│  DB_HOST=     │                 │  DB_HOST=    │
│   localhost   │                 │  prod-db.aws │
│  DB_DATABASE= │                 │  DB_DATABASE=│
│   my_app_dev  │                 │  my_app_prod │
└──────────────┘                 └──────────────┘
```

## The .env File

The `.env` file contains your environment-specific settings:

```env
# Application
APP_NAME="My Application"
APP_ENV=local
APP_KEY=base64:random_key_here
APP_DEBUG=true
APP_URL=http://localhost

# Database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=my_app
DB_USERNAME=root
DB_PASSWORD=secret

# Mail
MAIL_MAILER=smtp
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"  # Can reference other variables

# Third-party Services
STRIPE_KEY=sk_test_...
STRIPE_SECRET=pk_test_...
```

## The env() Helper

Access environment variables with the `env()` function:

```php
// Get environment variable
$debug = env('APP_DEBUG');

// With a default value
$host = env('DB_HOST', 'localhost');

// In configuration files
// config/database.php
return [
    'default' => env('DB_CONNECTION', 'sqlite'),
    'connections' => [
        'mysql' => [
            'host' => env('DB_HOST', '127.0.0.1'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
        ],
    ],
];
```

## Important: Only Use env() in Config Files

**Never** use `env()` directly in your application code:

```php
// ❌ Bad - Don't do this in controllers, models, etc.
public function index()
{
    $apiKey = env('API_KEY');  // May return null in production!
}

// ✅ Good - Use config() instead
public function index()
{
    $apiKey = config('services.api.key');
}

// And define it in config/services.php
return [
    'api' => [
        'key' => env('API_KEY'),
    ],
];
```

Why? Because in production, configuration is cached and `env()` calls outside config files return `null`.

## Environment Detection

Check the current environment:

```php
// Using App facade
use Illuminate\Support\Facades\App;

if (App::environment('local')) {
    // Running locally
}

if (App::environment(['local', 'staging'])) {
    // Local or staging
}

// Using the app() helper
if (app()->environment('production')) {
    // Production environment
}

// Using config
if (config('app.env') === 'local') {
    // Local environment
}
```

## The .env.example File

The `.env.example` file is a template that **should** be in version control:

```env
# .env.example - Committed to version control
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true

DB_CONNECTION=sqlite
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel
# DB_USERNAME=root
# DB_PASSWORD=
```

When someone clones your project, they copy `.env.example` to `.env` and fill in their values.

## Configuration Caching

In production, cache configuration for performance:

```bash
# Cache all configuration into a single file
php artisan config:cache

# Clear the cache
php artisan config:clear
```

**Important**: After caching, `env()` calls only work inside config files!

## Different Environments

You can have multiple environment files:

```
.env              # Default (development)
.env.local        # Local overrides
.env.staging      # Staging environment
.env.production   # Production (but usually set via server)
.env.testing      # Testing environment
```

Set which file to use:

```bash
# Use a specific environment file
APP_ENV=staging php artisan migrate
```

## Resources

- [Configuration - Environment](https://laravel.com/docs/12.x/configuration#environment-configuration) — Official guide to environment configuration

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*