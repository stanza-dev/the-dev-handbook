---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-understanding-migrations"
---

# Understanding Database Migrations

Migrations are like version control for your database schema. They allow your team to define and share the application's database schema definition.

## Why Migrations?

### Without Migrations

```
"Hey, I added a column to the users table."
"What's it called? What type?"
"Just run this SQL: ALTER TABLE users ADD COLUMN..."
"Did you update the staging server too?"
🤯
```

### With Migrations

```
git pull
php artisan migrate
✅ Database updated!
```

## Migration Benefits

| Benefit | Description |
|---------|-------------|
| **Version Control** | Schema changes are tracked in Git |
| **Team Sync** | Everyone has the same database structure |
| **Environment Parity** | Dev, staging, and production stay in sync |
| **Rollback** | Undo changes if something goes wrong |
| **Database Agnostic** | Same migrations work on MySQL, PostgreSQL, SQLite |

## Creating Migrations

```bash
# Create a migration
php artisan make:migration create_posts_table

# With model
php artisan make:model Post -m

# For modifying existing table
php artisan make:migration add_views_to_posts_table --table=posts
```

Migration files are created in `database/migrations/`:

```
database/migrations/
├── 2024_01_01_000000_create_users_table.php
├── 2024_01_01_000001_create_cache_table.php
├── 2024_01_15_143022_create_posts_table.php
└── 2024_01_20_091500_add_views_to_posts_table.php
```

## Migration Structure

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
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('body');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

## Running Migrations

```bash
# Run all pending migrations
php artisan migrate

# See status
php artisan migrate:status

# Output:
# Migration name ............................. Batch / Status
# 2024_01_01_000000_create_users_table ............... [1] Ran
# 2024_01_15_143022_create_posts_table ............... Pending
```

## Rolling Back

```bash
# Rollback last batch
php artisan migrate:rollback

# Rollback last 3 migrations
php artisan migrate:rollback --step=3

# Rollback all migrations
php artisan migrate:reset

# Rollback all and re-run
php artisan migrate:refresh

# Drop all tables and re-run
php artisan migrate:fresh

# Fresh with seeding
php artisan migrate:fresh --seed
```

## The `up()` and `down()` Methods

```php
public function up(): void
{
    // Create, modify, or add to the database
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        // ...
    });
}

public function down(): void
{
    // Reverse whatever up() did
    // Should return database to previous state
    Schema::dropIfExists('posts');
}
```

### Why `down()` Matters

```php
// Bad: down() doesn't reverse up()
public function up(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->string('subtitle')->nullable();
        $table->integer('views')->default(0);
    });
}

public function down(): void
{
    // ❌ Missing: doesn't remove the columns!
}

// Good: down() properly reverses up()
public function down(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->dropColumn(['subtitle', 'views']);
    });
}
```

## Migration Tips

### 1. Never Edit Deployed Migrations

```php
// ❌ Don't modify migrations that have run in production
// ✅ Create a new migration to make changes
```

### 2. Use Descriptive Names

```bash
# Good
php artisan make:migration add_published_at_to_posts_table
php artisan make:migration create_post_category_pivot_table
php artisan make:migration drop_legacy_users_table

# Bad
php artisan make:migration update_posts
php artisan make:migration changes
```

### 3. Keep Migrations Small

```php
// ✅ One logical change per migration
// ❌ Don't combine unrelated changes
```

## Resources

- [Database Migrations](https://laravel.com/docs/12.x/migrations) — Official migrations documentation

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*