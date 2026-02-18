---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-database-setup"
---

# Setting Up the Database

Now let's create the database structure for our task manager. We'll use Laravel migrations to define our tables.

## Understanding Migrations

Migrations are like **version control for your database**. They allow you to:

- Define database schema in PHP code
- Share database changes with your team
- Roll back changes if something goes wrong
- Track all changes over time

## Creating the Categories Migration

First, let's create the categories table:

```bash
php artisan make:migration create_categories_table
```

Open `database/migrations/xxxx_xx_xx_create_categories_table.php`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();  // Auto-incrementing ID
            
            // Foreign key to users table
            $table->foreignId('user_id')
                  ->constrained()  // References users.id
                  ->cascadeOnDelete();  // Delete categories when user is deleted
            
            $table->string('name');  // Category name
            $table->string('color', 7)->default('#6366f1');  // Hex color code
            
            $table->timestamps();  // created_at and updated_at
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('categories');
    }
};
```

### Understanding the Schema

| Method | Description |
|--------|-------------|
| `$table->id()` | Creates auto-incrementing `id` column |
| `$table->foreignId('user_id')` | Creates `user_id` column for foreign key |
| `->constrained()` | Adds foreign key constraint to `users.id` |
| `->cascadeOnDelete()` | Deletes categories when user is deleted |
| `$table->string('name')` | VARCHAR column for category name |
| `$table->string('color', 7)` | VARCHAR(7) for hex color (#RRGGBB) |
| `$table->timestamps()` | Adds `created_at` and `updated_at` columns |

## Creating the Tasks Migration

Now create the tasks table:

```bash
php artisan make:migration create_tasks_table
```

Edit `database/migrations/xxxx_xx_xx_create_tasks_table.php`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            
            // Foreign key to users table
            $table->foreignId('user_id')
                  ->constrained()
                  ->cascadeOnDelete();
            
            // Foreign key to categories (nullable - task may not have category)
            $table->foreignId('category_id')
                  ->nullable()  // Task doesn't require a category
                  ->constrained()
                  ->nullOnDelete();  // Set to null if category is deleted
            
            // Task details
            $table->string('title');
            $table->text('description')->nullable();
            $table->boolean('completed')->default(false);
            $table->date('due_date')->nullable();
            $table->enum('priority', ['low', 'medium', 'high'])->default('medium');
            
            $table->timestamps();
            
            // Index for faster queries
            $table->index(['user_id', 'completed']);
            $table->index(['user_id', 'due_date']);
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('tasks');
    }
};
```

### New Concepts Explained

| Method | Description |
|--------|-------------|
| `->nullable()` | Column can be NULL |
| `->nullOnDelete()` | Set to NULL when related record is deleted |
| `$table->text()` | TEXT column for longer content |
| `$table->boolean()` | BOOLEAN/TINYINT column |
| `$table->date()` | DATE column (no time) |
| `$table->enum()` | ENUM column with specific allowed values |
| `$table->index()` | Creates database index for faster queries |

## Running the Migrations

Apply the migrations to create the tables:

```bash
php artisan migrate
```

You should see:

```
   INFO  Running migrations.

  2024_01_15_100000_create_categories_table .......... 5.42ms DONE
  2024_01_15_100001_create_tasks_table ............... 8.31ms DONE
```

## Checking the Database

Use Tinker to verify the tables exist:

```bash
php artisan tinker
```

```php
>>> Schema::hasTable('categories')
=> true

>>> Schema::hasTable('tasks')
=> true

>>> Schema::getColumnListing('tasks')
=> [
     "id",
     "user_id",
     "category_id",
     "title",
     "description",
     "completed",
     "due_date",
     "priority",
     "created_at",
     "updated_at",
   ]
```

## Migration Commands Reference

```bash
# Run all pending migrations
php artisan migrate

# Rollback the last batch of migrations
php artisan migrate:rollback

# Rollback all migrations
php artisan migrate:reset

# Rollback and re-run all migrations
php artisan migrate:refresh

# Drop all tables and re-run migrations
php artisan migrate:fresh

# Check migration status
php artisan migrate:status
```

## Database Diagram

Our complete database structure:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│     users       │     │   categories    │     │     tasks       │
├─────────────────┤     ├─────────────────┤     ├─────────────────┤
│ id (PK)         │◄────│ user_id (FK)    │     │ id (PK)         │
│ name            │     │ id (PK)         │◄────│ category_id (FK)│
│ email           │     │ name            │     │ user_id (FK)    │────►
│ password        │     │ color           │     │ title           │
│ created_at      │     │ created_at      │     │ description     │
│ updated_at      │     │ updated_at      │     │ completed       │
└─────────────────┘     └─────────────────┘     │ due_date        │
                                                │ priority        │
                                                │ created_at      │
                                                │ updated_at      │
                                                └─────────────────┘
```

Next, we'll create Eloquent models to interact with these tables!

## Resources

- [Database Migrations](https://laravel.com/docs/12.x/migrations) — Complete guide to Laravel migrations

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*