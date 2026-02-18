---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-observers"
---

# Creating and Using Observers

Observers are classes dedicated to listening to model events. They're ideal when a model has many event listeners or when the logic is complex.

## Why Use Observers?

```php
// ❌ Model becomes cluttered with event logic
class Post extends Model
{
    protected static function booted(): void
    {
        static::creating(function ($post) { /* 20 lines */ });
        static::created(function ($post) { /* 15 lines */ });
        static::updating(function ($post) { /* 25 lines */ });
        static::deleting(function ($post) { /* 30 lines */ });
    }
}

// ✅ Clean model, logic in dedicated observer class
class Post extends Model
{
    // Clean!
}

class PostObserver
{
    // All event logic organized here
}
```

## Creating an Observer

```bash
php artisan make:observer PostObserver --model=Post
```

This creates `app/Observers/PostObserver.php`:

```php
<?php

namespace App\Observers;

use App\Models\Post;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\Log;

class PostObserver
{
    /**
     * Handle the Post "creating" event.
     * Fires BEFORE the model is saved for the first time.
     */
    public function creating(Post $post): void
    {
        // Generate slug
        $post->slug = Str::slug($post->title);
        
        // Set author if not set
        $post->user_id = $post->user_id ?? auth()->id();
        
        // Generate UUID
        $post->uuid = Str::uuid();
    }

    /**
     * Handle the Post "created" event.
     * Fires AFTER the model is saved for the first time.
     */
    public function created(Post $post): void
    {
        Log::info('Post created', ['id' => $post->id, 'title' => $post->title]);
        
        // Notify followers
        $post->user->followers->each(function ($follower) use ($post) {
            $follower->notify(new NewPostNotification($post));
        });
    }

    /**
     * Handle the Post "updating" event.
     * Fires BEFORE an existing model is saved.
     */
    public function updating(Post $post): void
    {
        // Regenerate slug if title changed
        if ($post->isDirty('title')) {
            $post->slug = Str::slug($post->title);
        }
    }

    /**
     * Handle the Post "updated" event.
     * Fires AFTER an existing model is saved.
     */
    public function updated(Post $post): void
    {
        // Clear cache
        Cache::forget("post:{$post->id}");
        
        Log::info('Post updated', ['id' => $post->id]);
    }

    /**
     * Handle the Post "saving" event.
     * Fires BEFORE any save (create or update).
     */
    public function saving(Post $post): void
    {
        // Sanitize content
        $post->body = strip_tags($post->body, '<p><br><strong><em><ul><li>');
    }

    /**
     * Handle the Post "saved" event.
     * Fires AFTER any save (create or update).
     */
    public function saved(Post $post): void
    {
        // Index for search
        SearchIndex::update($post);
    }

    /**
     * Handle the Post "deleting" event.
     * Return false to cancel deletion.
     */
    public function deleting(Post $post): bool
    {
        // Prevent deletion if post has comments
        if ($post->comments()->exists()) {
            return false;
        }
        
        return true;
    }

    /**
     * Handle the Post "deleted" event.
     */
    public function deleted(Post $post): void
    {
        // Clean up associated files
        if ($post->featured_image) {
            Storage::delete($post->featured_image);
        }
        
        // Remove from search index
        SearchIndex::remove($post);
        
        Log::info('Post deleted', ['id' => $post->id]);
    }

    /**
     * Handle the Post "restored" event (soft deletes).
     */
    public function restored(Post $post): void
    {
        // Re-index for search
        SearchIndex::update($post);
    }

    /**
     * Handle the Post "forceDeleted" event.
     */
    public function forceDeleted(Post $post): void
    {
        // Permanently delete all comments
        $post->comments()->forceDelete();
    }
}
```

## Registering Observers

### Method 1: Using the `#[ObservedBy]` Attribute (Laravel 11+)

```php
<?php

namespace App\Models;

use App\Observers\PostObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;
use Illuminate\Database\Eloquent\Model;

#[ObservedBy([PostObserver::class])]
class Post extends Model
{
    // ...
}
```

### Method 2: In the Model's `booted()` Method

```php
<?php

namespace App\Models;

use App\Observers\PostObserver;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    protected static function booted(): void
    {
        static::observe(PostObserver::class);
    }
}
```

### Method 3: In a Service Provider

```php
// app/Providers/AppServiceProvider.php
use App\Models\Post;
use App\Observers\PostObserver;

public function boot(): void
{
    Post::observe(PostObserver::class);
}
```

## Preventing Infinite Loops

Be careful when updating models in observers:

```php
// ❌ DANGER: Infinite loop!
public function updating(Post $post): void
{
    $post->view_count++;  // This triggers updating again!
}

// ✅ SAFE: Use saveQuietly()
public function updated(Post $post): void
{
    $post->timestamps = false;  // Don't update updated_at
    $post->saveQuietly();       // Skip events
    $post->timestamps = true;
}

// ✅ SAFE: Use DB query directly
public function updated(Post $post): void
{
    DB::table('posts')
        ->where('id', $post->id)
        ->update(['last_activity' => now()]);
}
```

## Conditional Logic in Observers

```php
public function updating(Post $post): void
{
    // Only regenerate slug if title actually changed
    if ($post->isDirty('title')) {
        $post->slug = Str::slug($post->title);
    }
    
    // Check what the old value was
    if ($post->isDirty('status')) {
        $oldStatus = $post->getOriginal('status');
        $newStatus = $post->status;
        
        if ($oldStatus === 'draft' && $newStatus === 'published') {
            $post->published_at = now();
        }
    }
}
```

## Observer Best Practices

1. **Keep observers focused** - One observer per model
2. **Don't make HTTP calls** in observers - Use queued jobs instead
3. **Be careful with relationships** - They may not be loaded
4. **Test your observers** - They can cause subtle bugs
5. **Document side effects** - Make it clear what happens when

## Resources

- [Observers](https://laravel.com/docs/12.x/eloquent#observers) — Official documentation on Eloquent observers

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*