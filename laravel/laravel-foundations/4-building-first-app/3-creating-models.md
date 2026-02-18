---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-creating-models"
---

# Creating Eloquent Models

Models are the heart of your Laravel application. They represent your data, define relationships, and contain business logic.

## Creating the Category Model

Generate the model:

```bash
php artisan make:model Category
```

Edit `app/Models/Category.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Category extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'user_id',
        'name',
        'color',
    ];

    /**
     * Get the user that owns the category.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the tasks in this category.
     */
    public function tasks(): HasMany
    {
        return $this->hasMany(Task::class);
    }
}
```

### Understanding the Model

- **`$fillable`**: Lists columns that can be mass-assigned (security protection)
- **`user()`**: Defines the "belongs to" relationship with User
- **`tasks()`**: Defines the "has many" relationship with Task

## Creating the Task Model

Generate the model:

```bash
php artisan make:model Task
```

Edit `app/Models/Task.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Builder;

class Task extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'user_id',
        'category_id',
        'title',
        'description',
        'completed',
        'due_date',
        'priority',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'completed' => 'boolean',
            'due_date' => 'date',
        ];
    }

    /**
     * Get the user that owns the task.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the category of the task.
     */
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    /**
     * Scope: Only completed tasks.
     */
    public function scopeCompleted(Builder $query): Builder
    {
        return $query->where('completed', true);
    }

    /**
     * Scope: Only incomplete tasks.
     */
    public function scopeIncomplete(Builder $query): Builder
    {
        return $query->where('completed', false);
    }

    /**
     * Scope: Tasks due today.
     */
    public function scopeDueToday(Builder $query): Builder
    {
        return $query->whereDate('due_date', today());
    }

    /**
     * Scope: Overdue tasks.
     */
    public function scopeOverdue(Builder $query): Builder
    {
        return $query->where('completed', false)
                     ->whereDate('due_date', '<', today());
    }

    /**
     * Check if the task is overdue.
     */
    public function isOverdue(): bool
    {
        return !$this->completed 
            && $this->due_date 
            && $this->due_date->isPast();
    }
}
```

### New Concepts Explained

#### Attribute Casting

```php
protected function casts(): array
{
    return [
        'completed' => 'boolean',  // Casts 0/1 to true/false
        'due_date' => 'date',       // Casts to Carbon date object
    ];
}
```

Casting automatically converts database values to PHP types.

#### Query Scopes

Scopes are reusable query constraints:

```php
// Define scope
public function scopeCompleted(Builder $query): Builder
{
    return $query->where('completed', true);
}

// Use scope
Task::completed()->get();           // All completed tasks
Task::incomplete()->dueToday()->get();  // Incomplete tasks due today
$user->tasks()->overdue()->get();   // User's overdue tasks
```

#### Custom Methods

```php
public function isOverdue(): bool
{
    return !$this->completed 
        && $this->due_date 
        && $this->due_date->isPast();
}

// Usage
if ($task->isOverdue()) {
    // Show warning
}
```

## Updating the User Model

Add relationships to the existing User model. Edit `app/Models/User.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }

    /**
     * Get the tasks for the user.
     */
    public function tasks(): HasMany
    {
        return $this->hasMany(Task::class);
    }

    /**
     * Get the categories for the user.
     */
    public function categories(): HasMany
    {
        return $this->hasMany(Category::class);
    }
}
```

## Testing Models with Tinker

Let's test our models:

```bash
php artisan tinker
```

```php
// Create a test user first
>>> $user = User::create(['name' => 'John', 'email' => 'john@example.com', 'password' => bcrypt('password')])

// Create a category
>>> $category = $user->categories()->create(['name' => 'Work', 'color' => '#ef4444'])

// Create tasks
>>> $task = $user->tasks()->create([
...     'title' => 'Complete Laravel course',
...     'description' => 'Finish all lessons and challenges',
...     'category_id' => $category->id,
...     'due_date' => now()->addDays(7),
...     'priority' => 'high'
... ])

// Query tasks
>>> $user->tasks()->count()
=> 1

>>> $user->tasks()->incomplete()->get()
=> Tasks collection...

>>> $task->category->name
=> "Work"

>>> $task->isOverdue()
=> false
```

## Relationship Summary

```php
// From User
$user->tasks;              // All user's tasks
$user->tasks()->create();  // Create task for user
$user->categories;         // All user's categories

// From Task
$task->user;               // Task's owner
$task->category;           // Task's category
$task->isOverdue();        // Check if overdue

// From Category
$category->user;           // Category's owner
$category->tasks;          // Tasks in this category
```

Now let's create the routes and controllers!

## Resources

- [Eloquent Models](https://laravel.com/docs/12.x/eloquent) — Complete guide to Eloquent ORM
- [Eloquent Relationships](https://laravel.com/docs/12.x/eloquent-relationships) — Understanding Eloquent relationships

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*