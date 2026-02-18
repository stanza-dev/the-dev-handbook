---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-model-events"
---

# Model Events and Observers

Eloquent models fire events during their lifecycle, allowing you to hook into key moments like creating, updating, or deleting.

## Model Events

Eloquent fires these events:

| Event | When Fired |
|-------|------------|
| `retrieved` | After model is retrieved from database |
| `creating` | Before model is created |
| `created` | After model is created |
| `updating` | Before model is updated |
| `updated` | After model is updated |
| `saving` | Before model is created OR updated |
| `saved` | After model is created OR updated |
| `deleting` | Before model is deleted |
| `deleted` | After model is deleted |
| `trashed` | After model is soft deleted |
| `forceDeleting` | Before force deleting |
| `forceDeleted` | After force deleting |
| `restoring` | Before restoring soft deleted |
| `restored` | After restoring soft deleted |
| `replicating` | When model is replicated |

## Using Closures in Models

```php
class User extends Model
{
    protected static function booted(): void
    {
        static::creating(function (User $user) {
            $user->uuid = Str::uuid();
        });

        static::created(function (User $user) {
            $user->profile()->create();
        });

        static::deleting(function (User $user) {
            $user->posts()->delete();
        });
    }
}
```

## Creating Observers

```bash
php artisan make:observer UserObserver --model=User
```

This creates `app/Observers/UserObserver.php`:

```php
<?php

namespace App\Observers;

use App\Models\User;

class UserObserver
{
    public function creating(User $user): void
    {
        $user->uuid = Str::uuid();
    }

    public function created(User $user): void
    {
        $user->profile()->create(['bio' => '']);
        $user->notify(new WelcomeNotification());
    }

    public function updated(User $user): void
    {
        if ($user->isDirty('email')) {
            $user->email_verified_at = null;
            $user->saveQuietly();  // Don't trigger events
            $user->sendEmailVerificationNotification();
        }
    }

    public function deleted(User $user): void
    {
        Log::info('User deleted', ['user_id' => $user->id]);
    }

    public function forceDeleted(User $user): void
    {
        Storage::delete($user->avatar_path);
    }
}
```

## Registering Observers

In `AppServiceProvider`:

```php
use App\Models\User;
use App\Observers\UserObserver;

public function boot(): void
{
    User::observe(UserObserver::class);
}
```

Or using the attribute:

```php
use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy(UserObserver::class)]
class User extends Model
{
    // ...
}
```

## Preventing Infinite Loops

Use `saveQuietly()` to avoid triggering events:

```php
public function updated(User $user): void
{
    // This would cause infinite loop:
    // $user->update(['last_activity' => now()]);

    // Use saveQuietly instead:
    $user->last_activity = now();
    $user->saveQuietly();
}
```

## Conditional Logic

```php
public function saving(User $user): bool|void
{
    // Return false to cancel the operation
    if ($user->isBanned()) {
        return false;
    }
}

public function deleting(User $user): bool|void
{
    // Prevent deletion if user has active subscriptions
    if ($user->hasActiveSubscription()) {
        return false;
    }
}
```

## Complete Observer Example

```php
<?php

namespace App\Observers;

use App\Models\Post;
use App\Jobs\GeneratePostSlug;
use App\Jobs\NotifySubscribers;
use Illuminate\Support\Str;

class PostObserver
{
    public function creating(Post $post): void
    {
        $post->user_id ??= auth()->id();
        $post->slug ??= Str::slug($post->title);
    }

    public function created(Post $post): void
    {
        cache()->forget('posts:count');

        if ($post->published_at) {
            NotifySubscribers::dispatch($post);
        }
    }

    public function updating(Post $post): void
    {
        if ($post->isDirty('title')) {
            $post->slug = Str::slug($post->title);
        }
    }

    public function updated(Post $post): void
    {
        cache()->forget("post:{$post->id}");
        cache()->forget('posts:latest');
    }

    public function deleted(Post $post): void
    {
        cache()->forget("post:{$post->id}");
        cache()->forget('posts:count');
    }
}
```

## Resources

- [Eloquent Observers](https://laravel.com/docs/12.x/eloquent#observers) — Official documentation on Eloquent observers

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*