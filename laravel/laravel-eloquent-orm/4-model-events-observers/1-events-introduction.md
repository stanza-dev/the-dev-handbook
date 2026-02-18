---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-model-events-introduction"
---

# Introduction to Model Events

Eloquent models fire several events during their lifecycle, allowing you to hook into key moments like creating, updating, or deleting records. This enables you to automate actions and keep your controllers clean.

## Available Model Events

Eloquent dispatches these events during a model's lifecycle:

```
┌─────────────────────────────────────────────────────────────────┐
│                    MODEL LIFECYCLE EVENTS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CREATING ─────► CREATED                                        │
│  (before save)   (after save)                                   │
│                                                                  │
│  UPDATING ─────► UPDATED                                        │
│  (before save)   (after save)                                   │
│                                                                  │
│  SAVING ───────► SAVED                                          │
│  (before any     (after any save - create or update)            │
│   save)                                                          │
│                                                                  │
│  DELETING ─────► DELETED                                        │
│  (before delete) (after delete)                                 │
│                                                                  │
│  TRASHED ─────── (soft delete only, after deleted)              │
│                                                                  │
│  RESTORING ────► RESTORED                                       │
│  (before restore)(after restore from soft delete)               │
│                                                                  │
│  FORCE_DELETING ► FORCE_DELETED                                 │
│  (permanent      (after permanent delete)                       │
│   delete)                                                        │
│                                                                  │
│  REPLICATING                                                    │
│  (when using replicate())                                       │
│                                                                  │
│  RETRIEVED                                                      │
│  (after fetching from database)                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Event Sequence

### Creating a New Model

```
$post = Post::create([...]);

1. saving     (before insert)
2. creating   (before insert, only for new records)
3. INSERT INTO posts ...
4. created    (after insert)
5. saved      (after insert)
```

### Updating an Existing Model

```
$post->update([...]);

1. saving     (before update)
2. updating   (before update, only for existing records)
3. UPDATE posts SET ...
4. updated    (after update)
5. saved      (after update)
```

## Defining Events with Closures

The simplest way to listen to model events is using closures in the `booted()` method:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Str;

class Post extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        // Generate slug before creating
        static::creating(function (Post $post) {
            $post->slug = Str::slug($post->title);
        });

        // Log when a post is created
        static::created(function (Post $post) {
            logger()->info('New post created', ['id' => $post->id]);
        });

        // Prevent deletion if post has comments
        static::deleting(function (Post $post) {
            if ($post->comments()->exists()) {
                return false;  // Cancels the delete!
            }
        });

        // Clean up after deletion
        static::deleted(function (Post $post) {
            // Delete associated files
            Storage::delete($post->featured_image);
        });
    }
}
```

## Common Use Cases

### Auto-generating Values

```php
static::creating(function (Post $post) {
    // Generate UUID
    $post->uuid = Str::uuid();
    
    // Generate slug from title
    $post->slug = Str::slug($post->title);
    
    // Set default values
    $post->status = $post->status ?? 'draft';
    
    // Set the author
    $post->user_id = $post->user_id ?? auth()->id();
});
```

### Maintaining Related Data

```php
static::updating(function (Post $post) {
    // Update slug if title changed
    if ($post->isDirty('title')) {
        $post->slug = Str::slug($post->title);
    }
});

static::deleted(function (Post $post) {
    // Delete all comments when post is deleted
    $post->comments()->delete();
    
    // Or detach tags
    $post->tags()->detach();
});
```

### Validation / Prevention

```php
static::deleting(function (User $user) {
    // Prevent deleting admin users
    if ($user->is_admin) {
        throw new \Exception('Cannot delete admin users');
    }
    
    // Or return false to silently cancel
    if ($user->posts()->exists()) {
        return false;
    }
});
```

### Caching

```php
static::saved(function (Setting $setting) {
    // Clear settings cache when any setting changes
    Cache::forget('app_settings');
});

static::deleted(function (Setting $setting) {
    Cache::forget('app_settings');
});
```

## Canceling Events

Return `false` from `creating`, `updating`, `saving`, `deleting`, or `restoring` to cancel:

```php
static::creating(function (Post $post) {
    if ($post->containsSpam()) {
        return false;  // Prevents creation!
    }
});
```

## Important Notes

### Mass Updates Don't Fire Events

```php
// ❌ Events NOT fired!
Post::where('published', false)->update(['status' => 'draft']);

// ✅ Events fired for each model
Post::where('published', false)->get()->each->update(['status' => 'draft']);
```

### Quiet Operations

Skip events when needed:

```php
// Save without firing events
$post->saveQuietly();

// Delete without events
$post->deleteQuietly();

// Within a closure - no events fired
Post::withoutEvents(function () {
    Post::where('old', true)->delete();
});
```

## Resources

- [Eloquent Events](https://laravel.com/docs/12.x/eloquent#events) — Official documentation on Eloquent model events

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*