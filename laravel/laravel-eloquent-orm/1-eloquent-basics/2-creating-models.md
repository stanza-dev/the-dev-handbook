---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-creating-models"
---

# Creating Eloquent Models

Learn how to generate models, configure them, and customize Eloquent's conventions for your specific needs.

## Generating Models

Use Artisan to create models:

```bash
# Basic model
php artisan make:model Post

# Model with migration
php artisan make:model Post -m

# Model with migration, factory, seeder, and controller
php artisan make:model Post -mfsc

# Model with all related files
php artisan make:model Post --all

# API resource controller
php artisan make:model Post -mfscr --api
```

## Basic Model Structure

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasFactory;
}
```

## Customizing Table Names

Override the default table name:

```php
class Post extends Model
{
    /**
     * The table associated with the model.
     */
    protected $table = 'blog_posts';
}
```

## Customizing Primary Keys

```php
class Post extends Model
{
    /**
     * The primary key for the model.
     */
    protected $primaryKey = 'post_id';

    /**
     * Indicates if the primary key is auto-incrementing.
     */
    public $incrementing = false;

    /**
     * The data type of the primary key.
     */
    protected $keyType = 'string';  // For UUIDs
}
```

### Using UUIDs as Primary Keys

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;

class Post extends Model
{
    use HasUuids;

    // Eloquent automatically generates UUIDs
}

// Or use ULIDs (Universally Unique Lexicographically Sortable Identifier)
use Illuminate\Database\Eloquent\Concerns\HasUlids;

class Post extends Model
{
    use HasUlids;
}
```

## Timestamps Configuration

```php
class Post extends Model
{
    /**
     * Disable timestamps entirely.
     */
    public $timestamps = false;

    /**
     * Customize timestamp column names.
     */
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'last_updated';
}
```

## Database Connection

Use a different database connection:

```php
class Post extends Model
{
    /**
     * The database connection for the model.
     */
    protected $connection = 'mysql_secondary';
}
```

## Default Attribute Values

```php
class Post extends Model
{
    /**
     * Default values for attributes.
     */
    protected $attributes = [
        'status' => 'draft',
        'is_featured' => false,
    ];
}

// Usage
$post = new Post;
$post->status;  // 'draft' (default)
```

## Strict Mode (Recommended)

Enable strict mode to catch issues early:

```php
// In AppServiceProvider boot()
use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    // Prevent lazy loading (use eager loading instead)
    Model::preventLazyLoading(! $this->app->isProduction());

    // Prevent silently discarding attributes
    Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());

    // Prevent accessing missing attributes
    Model::preventAccessingMissingAttributes(! $this->app->isProduction());
}
```

## Mass Assignment Protection

Control which attributes can be mass-assigned:

```php
class Post extends Model
{
    /**
     * Attributes that ARE mass assignable.
     */
    protected $fillable = [
        'title',
        'body',
        'category_id',
    ];
}

// Or specify which are NOT mass assignable
class Post extends Model
{
    /**
     * Attributes that are NOT mass assignable.
     */
    protected $guarded = [
        'id',
        'is_admin',
    ];

    // Or guard nothing (be careful!)
    protected $guarded = [];
}
```

### Why Mass Assignment Protection?

```php
// Without protection, malicious users could do:
// POST /users with {"name": "John", "is_admin": true}

User::create($request->all());  // is_admin would be set!

// With $fillable = ['name', 'email']:
User::create($request->all());  // is_admin is ignored
```

## Complete Model Example

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use HasFactory, SoftDeletes;

    /**
     * The attributes that are mass assignable.
     */
    protected $fillable = [
        'title',
        'slug',
        'body',
        'published_at',
        'user_id',
        'category_id',
    ];

    /**
     * The attributes that should be hidden for arrays/JSON.
     */
    protected $hidden = [
        'internal_notes',
    ];

    /**
     * Get the attributes that should be cast.
     */
    protected function casts(): array
    {
        return [
            'published_at' => 'datetime',
            'is_featured' => 'boolean',
            'metadata' => 'array',
        ];
    }
}
```

## Resources

- [Eloquent Model Conventions](https://laravel.com/docs/12.x/eloquent#eloquent-model-conventions) — Official documentation on model conventions

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*