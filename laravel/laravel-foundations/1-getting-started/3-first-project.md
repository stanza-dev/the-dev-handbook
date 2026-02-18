---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-creating-first-project"
---

# Creating Your First Laravel Project

Let's create your first Laravel project and explore its structure. This hands-on experience will help you understand how Laravel organizes code.

## Creating a New Project

With PHP, Composer, and the Laravel installer ready:

```bash
# Create a new Laravel project
laravel new my-first-app
```

The installer will prompt you for several options:

1. **Starter Kit**: Choose "No starter kit" for now (we'll explore these later)
2. **Testing Framework**: Choose "Pest" or "PHPUnit"
3. **Database**: Choose "SQLite" for simplicity
4. **Git Repository**: Yes to initialize Git

## Project Structure

Laravel generates this structure:

```
my-first-app/
├── app/                    # Application code
│   ├── Http/
│   │   ├── Controllers/    # Request handlers
│   │   └── Middleware/     # Request filters
│   ├── Models/             # Eloquent models
│   └── Providers/          # Service providers
├── bootstrap/              # Framework bootstrap
├── config/                 # Configuration files
├── database/
│   ├── factories/          # Model factories
│   ├── migrations/         # Database migrations
│   └── seeders/            # Database seeders
├── public/                 # Web root (index.php)
├── resources/
│   ├── css/                # CSS files
│   ├── js/                 # JavaScript files
│   └── views/              # Blade templates
├── routes/
│   ├── web.php             # Web routes
│   └── api.php             # API routes
├── storage/                # Logs, cache, uploads
├── tests/                  # Test files
├── vendor/                 # Composer packages
├── .env                    # Environment variables
├── artisan                 # CLI tool
├── composer.json           # PHP dependencies
└── package.json            # Node dependencies
```

## Understanding Key Directories

### app/

Contains your application code:

```php
// app/Models/User.php - Eloquent model
class User extends Authenticatable
{
    // Model definition
}

// app/Http/Controllers/UserController.php
class UserController extends Controller
{
    public function index()
    {
        return view('users.index');
    }
}
```

### config/

Configuration files for every aspect of Laravel:

```
config/
├── app.php         # Application settings
├── database.php    # Database connections
├── mail.php        # Email configuration
├── cache.php       # Caching settings
└── ...             # Many more
```

### routes/

Defines how URLs map to controllers:

```php
// routes/web.php
Route::get('/', function () {
    return view('welcome');
});

Route::get('/users', [UserController::class, 'index']);
```

### resources/views/

Blade templates for HTML rendering:

```blade
<!-- resources/views/welcome.blade.php -->
<html>
    <head>
        <title>Laravel</title>
    </head>
    <body>
        <h1>Welcome to Laravel!</h1>
    </body>
</html>
```

## Starting the Development Server

Laravel includes a development server:

```bash
cd my-first-app

# Start all services (server, queue, vite)
composer run dev

# Or just the PHP server
php artisan serve
```

You'll see:

```
   INFO  Server running on [http://127.0.0.1:8000].

  Press Ctrl+C to stop the server
```

Open **http://127.0.0.1:8000** in your browser to see the Laravel welcome page!

## The .env File

Laravel uses environment variables for configuration:

```env
# .env
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:...
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=sqlite
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel
# DB_USERNAME=root
# DB_PASSWORD=

MAIL_MAILER=log
```

**Important**: Never commit `.env` to version control. Use `.env.example` as a template.

## First Database Migration

Laravel comes with migrations for users and sessions:

```bash
php artisan migrate
```

Output:

```
   INFO  Running migrations.

  0001_01_01_000000_create_users_table ............... 12.45ms DONE
  0001_01_01_000001_create_cache_table ............... 3.22ms DONE
  0001_01_01_000002_create_jobs_table ................ 8.91ms DONE
```

This creates the SQLite database and sets up the default tables.

## Resources

- [Directory Structure](https://laravel.com/docs/12.x/structure) — Official documentation on Laravel's directory structure

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*