---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-modifying-tables"
---

# Modifying Existing Tables

As your application evolves, you'll need to modify existing database tables. Laravel makes this easy with migration commands for adding, modifying, and removing columns.

## Adding Columns

```bash
php artisan make:migration add_phone_to_users_table --table=users
```

```php
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('phone', 20)->nullable()->after('email');
        $table->date('birth_date')->nullable();
        $table->json('preferences')->nullable();
    });
}

public function down(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['phone', 'birth_date', 'preferences']);
    });
}
```

## Modifying Columns

First, install the doctrine/dbal package:

```bash
composer require doctrine/dbal
```

Then modify columns:

```php
public function up(): void
{
    Schema::table('posts', function (Blueprint $table) {
        // Change column type
        $table->text('title')->change();  // Was string, now text

        // Make nullable
        $table->string('subtitle')->nullable()->change();

        // Change default
        $table->integer('views')->default(0)->change();

        // Rename column
        $table->renameColumn('body', 'content');
    });
}

public function down(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->string('title', 255)->change();
        $table->string('subtitle')->nullable(false)->change();
        $table->integer('views')->default(null)->change();
        $table->renameColumn('content', 'body');
    });
}
```

## Dropping Columns

```php
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        // Drop single column
        $table->dropColumn('legacy_field');

        // Drop multiple columns
        $table->dropColumn(['old_field', 'unused_field']);
    });
}
```

### Dropping with Indexes/Foreign Keys

```php
public function up(): void
{
    Schema::table('posts', function (Blueprint $table) {
        // Drop foreign key first, then column
        $table->dropForeign(['user_id']);  // Drops posts_user_id_foreign
        $table->dropColumn('user_id');

        // Drop index then column
        $table->dropIndex(['status']);      // Drops posts_status_index
        $table->dropColumn('status');

        // Drop unique constraint
        $table->dropUnique(['email']);      // Drops posts_email_unique
    });
}
```

## Managing Indexes

```php
public function up(): void
{
    Schema::table('posts', function (Blueprint $table) {
        // Add indexes
        $table->index('slug');
        $table->unique('email');
        $table->index(['status', 'published_at'], 'posts_status_published_index');
    });
}

public function down(): void
{
    Schema::table('posts', function (Blueprint $table) {
        // Drop by column name (Laravel generates name)
        $table->dropIndex(['slug']);        // posts_slug_index
        $table->dropUnique(['email']);      // posts_email_unique

        // Or drop by explicit name
        $table->dropIndex('posts_status_published_index');
    });
}
```

## Renaming Tables

```php
public function up(): void
{
    Schema::rename('posts', 'articles');
}

public function down(): void
{
    Schema::rename('articles', 'posts');
}
```

## Checking Table/Column Existence

```php
public function up(): void
{
    // Check if table exists
    if (Schema::hasTable('users')) {
        // Modify table
    }

    // Check if column exists
    if (Schema::hasColumn('users', 'email')) {
        // Column exists
    }

    // Check multiple columns
    if (Schema::hasColumns('users', ['email', 'name'])) {
        // Both columns exist
    }
}
```

## Real-World Migration Examples

### Adding a Status Column with Enum

```php
public function up(): void
{
    Schema::table('orders', function (Blueprint $table) {
        $table->enum('status', ['pending', 'processing', 'shipped', 'delivered', 'cancelled'])
            ->default('pending')
            ->after('total');
    });
}
```

### Converting Nullable to Required

```php
public function up(): void
{
    // First, update existing null values
    DB::table('posts')
        ->whereNull('published_at')
        ->update(['published_at' => now()]);

    // Then make non-nullable
    Schema::table('posts', function (Blueprint $table) {
        $table->timestamp('published_at')->nullable(false)->change();
    });
}
```

### Adding Soft Deletes

```php
public function up(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->softDeletes();  // Adds deleted_at column
    });
}

public function down(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->dropSoftDeletes();  // Removes deleted_at column
    });
}
```

### Pivot Table for Many-to-Many

```php
public function up(): void
{
    Schema::create('post_tag', function (Blueprint $table) {
        $table->foreignId('post_id')->constrained()->cascadeOnDelete();
        $table->foreignId('tag_id')->constrained()->cascadeOnDelete();
        $table->primary(['post_id', 'tag_id']);
        $table->timestamps();
    });
}
```

## Resources

- [Modifying Columns](https://laravel.com/docs/12.x/migrations#modifying-columns) — Official documentation on modifying columns

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*