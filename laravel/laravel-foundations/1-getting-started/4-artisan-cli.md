---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-artisan-cli"
---

# The Artisan Command Line Interface

Artisan is Laravel's powerful command-line interface. It provides dozens of helpful commands for common development tasks, and you can create your own custom commands.

## Getting Started with Artisan

Run Artisan from your project root:

```bash
# List all available commands
php artisan list

# Get help for a specific command
php artisan help migrate
```

## Essential Artisan Commands

### Development Server

```bash
# Start the development server
php artisan serve

# Start on a different port
php artisan serve --port=8080

# Allow external connections
php artisan serve --host=0.0.0.0
```

### Code Generation

Artisan can generate boilerplate code:

```bash
# Create a controller
php artisan make:controller UserController

# Create a resource controller (with CRUD methods)
php artisan make:controller PostController --resource

# Create a model
php artisan make:model Post

# Create model with migration, factory, and seeder
php artisan make:model Post -mfs

# Create a migration
php artisan make:migration create_posts_table

# Create middleware
php artisan make:middleware EnsureUserIsAdmin

# Create a form request (validation)
php artisan make:request StorePostRequest

# Create a Blade component
php artisan make:component Alert
```

### Database Commands

```bash
# Run migrations
php artisan migrate

# Rollback last migration batch
php artisan migrate:rollback

# Rollback all and re-migrate
php artisan migrate:fresh

# Run with seeding
php artisan migrate:fresh --seed

# Run seeders
php artisan db:seed

# Check migration status
php artisan migrate:status
```

### Cache Management

```bash
# Clear all caches (recommended during development)
php artisan optimize:clear

# Clear specific caches
php artisan cache:clear     # Application cache
php artisan config:clear    # Configuration cache
php artisan route:clear     # Route cache
php artisan view:clear      # Compiled views

# Cache for production
php artisan optimize        # Cache config and routes
```

### The Interactive Shell (Tinker)

Tinker is a REPL for interacting with your Laravel application:

```bash
php artisan tinker
```

```php
>>> User::count()
=> 5

>>> User::create(['name' => 'John', 'email' => 'john@example.com', 'password' => bcrypt('secret')])
=> App\Models\User {#4567
     name: "John",
     email: "john@example.com",
     ...
   }

>>> User::where('name', 'John')->first()
=> App\Models\User {#4568 ...}
```

### Route Information

```bash
# List all routes
php artisan route:list

# Filter routes
php artisan route:list --path=api
php artisan route:list --name=user
```

### Queue Management

```bash
# Process queue jobs
php artisan queue:work

# Listen with auto-restart on code changes
php artisan queue:listen

# Process a single job
php artisan queue:work --once
```

## Command Shortcuts

Many commands have shortcuts:

```bash
# These are equivalent:
php artisan make:controller UserController
php artisan make:con UserController  # Partial match
```

## Creating Custom Commands

Generate a command:

```bash
php artisan make:command SendEmails
```

This creates `app/Console/Commands/SendEmails.php`:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     */
    protected $signature = 'mail:send {user} {--queue}';

    /**
     * The console command description.
     */
    protected $description = 'Send marketing emails to a user';

    /**
     * Execute the console command.
     */
    public function handle(): int
    {
        $userId = $this->argument('user');
        $shouldQueue = $this->option('queue');

        $this->info("Sending email to user {$userId}...");

        // Send email logic here...

        $this->info('Email sent successfully!');

        return Command::SUCCESS;
    }
}
```

Run your custom command:

```bash
php artisan mail:send 1 --queue
```

## Output and Interaction

Artisan provides helpful output methods:

```php
public function handle()
{
    // Output messages
    $this->info('This is informational.');      // Green
    $this->error('This is an error!');          // Red
    $this->warn('This is a warning.');          // Yellow
    $this->line('Plain text output.');          // Normal
    $this->newLine();                           // Empty line

    // Ask for input
    $name = $this->ask('What is your name?');

    // Confirmation
    if ($this->confirm('Do you wish to continue?')) {
        // ...
    }

    // Progress bar
    $users = User::all();
    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        // Process user...
        $bar->advance();
    }

    $bar->finish();
}
```

## Resources

- [Artisan Console](https://laravel.com/docs/12.x/artisan) — Complete guide to Laravel's Artisan CLI

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*