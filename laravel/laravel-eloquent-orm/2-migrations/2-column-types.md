---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-column-types"
---

# Migration Column Types and Modifiers

Laravel's Schema Builder provides a fluent API for defining database columns. Let's explore all the column types and modifiers available.

## Common Column Types

### Strings and Text

```php
$table->char('code', 4);           // Fixed-length string
$table->string('name', 100);       // VARCHAR(100), default 255
$table->text('description');       // TEXT
$table->mediumText('content');     // MEDIUMTEXT
$table->longText('body');          // LONGTEXT
```

### Numbers

```php
$table->integer('quantity');          // INTEGER
$table->tinyInteger('status');        // TINYINT
$table->smallInteger('order');        // SMALLINT
$table->mediumInteger('medium');      // MEDIUMINT
$table->bigInteger('views');          // BIGINT

$table->unsignedInteger('count');     // UNSIGNED INTEGER
$table->unsignedBigInteger('user_id'); // For foreign keys

$table->float('amount', 8, 2);        // FLOAT(8, 2)
$table->double('latitude', 15, 8);    // DOUBLE(15, 8)
$table->decimal('price', 10, 2);      // DECIMAL(10, 2) - exact
```

### Boolean

```php
$table->boolean('is_active');         // BOOLEAN (TINYINT(1))
```

### Dates and Times

```php
$table->date('birth_date');           // DATE
$table->time('alarm_time');           // TIME
$table->dateTime('published_at');     // DATETIME
$table->timestamp('verified_at');     // TIMESTAMP
$table->timestamps();                 // created_at & updated_at
$table->timestampTz('created_at');    // TIMESTAMP with timezone
$table->year('graduation_year');      // YEAR
```

### JSON

```php
$table->json('settings');             // JSON
$table->jsonb('settings');            // JSONB (PostgreSQL)
```

### Binary

```php
$table->binary('data');               // BLOB
```

### Special Types

```php
$table->id();                         // Auto-incrementing BIGINT primary key
$table->uuid('id');                   // UUID column
$table->ulid('id');                   // ULID column
$table->foreignId('user_id');         // UNSIGNED BIGINT (for foreign keys)
$table->ipAddress('visitor_ip');      // IP address (VARCHAR(45))
$table->macAddress('device_mac');     // MAC address
$table->enum('status', ['draft', 'published', 'archived']);
$table->set('options', ['a', 'b', 'c']);  // SET type
```

## Column Modifiers

Modifiers change column behavior:

```php
$table->string('email')->nullable();           // Allow NULL
$table->string('email')->nullable(false);      // NOT NULL (default)
$table->integer('votes')->default(0);          // Default value
$table->string('email')->unique();             // UNIQUE constraint
$table->integer('order')->unsigned();          // UNSIGNED
$table->string('title')->comment('Post title'); // Column comment
$table->timestamp('added_at')->useCurrent();   // Default CURRENT_TIMESTAMP
$table->timestamp('updated_at')->useCurrentOnUpdate(); // Update on change
```

### Positioning (MySQL)

```php
$table->string('city')->after('address');      // After specific column
$table->string('id')->first();                 // First column
```

### Invisible Columns (MySQL 8.0.23+)

```php
$table->string('secret')->invisible();         // Excluded from SELECT *
```

## Primary Keys

```php
// Auto-incrementing primary key
$table->id();                              // id BIGINT UNSIGNED AUTO_INCREMENT
$table->id('post_id');                     // Custom name

// Composite primary key
$table->primary(['user_id', 'post_id']);

// UUID primary key
$table->uuid('id')->primary();
```

## Indexes

```php
// Single column
$table->string('email')->unique();         // Unique index
$table->string('slug')->index();           // Regular index

// Multiple columns
$table->unique(['email', 'tenant_id']);    // Composite unique
$table->index(['status', 'created_at']);   // Composite index

// Named indexes
$table->index('email', 'users_email_index');

// Full-text index
$table->text('body');
$table->fullText('body');

// Spatial index
$table->point('location');
$table->spatialIndex('location');
```

## Foreign Keys

```php
// Method 1: Explicit foreign key
$table->unsignedBigInteger('user_id');
$table->foreign('user_id')
    ->references('id')
    ->on('users')
    ->onDelete('cascade');

// Method 2: Shorthand (recommended)
$table->foreignId('user_id')->constrained();

// With options
$table->foreignId('user_id')
    ->constrained()             // References users.id
    ->onUpdate('cascade')
    ->onDelete('cascade');

// Custom table/column
$table->foreignId('author_id')
    ->constrained('users', 'id')
    ->nullOnDelete();

// Nullable foreign key
$table->foreignId('category_id')
    ->nullable()
    ->constrained()
    ->nullOnDelete();  // Set to NULL when parent deleted
```

## Complete Migration Example

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->foreignId('category_id')->nullable()->constrained()->nullOnDelete();
            $table->string('title');
            $table->string('slug')->unique();
            $table->text('excerpt')->nullable();
            $table->longText('body');
            $table->string('featured_image')->nullable();
            $table->json('metadata')->nullable();
            $table->unsignedInteger('views')->default(0);
            $table->boolean('is_featured')->default(false);
            $table->timestamp('published_at')->nullable();
            $table->timestamps();
            $table->softDeletes();

            // Indexes
            $table->index(['published_at', 'is_featured']);
            $table->fullText('body');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

## Resources

- [Available Column Types](https://laravel.com/docs/12.x/migrations#available-column-types) — Complete list of column types

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*