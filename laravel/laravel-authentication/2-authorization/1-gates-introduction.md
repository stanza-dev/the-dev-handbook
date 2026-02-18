---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-gates-introduction"
---

# Authorization with Gates

Gates are closures that determine if a user is authorized to perform a given action. They're great for simple authorization checks.

## Defining Gates

Define gates in `AppServiceProvider` or a dedicated service provider:

```php
<?php

namespace App\Providers;

use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Simple gate
        Gate::define('access-admin', function (User $user) {
            return $user->is_admin;
        });

        // Gate with model parameter
        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id;
        });

        // Gate with response
        Gate::define('delete-post', function (User $user, Post $post) {
            if ($user->id === $post->user_id) {
                return true;
            }

            return Response::deny('You do not own this post.');
        });
    }
}
```

## Using Gates

### In Controllers

```php
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    public function update(Request $request, Post $post)
    {
        // Method 1: allows()
        if (Gate::allows('update-post', $post)) {
            // User can update
        }

        // Method 2: denies()
        if (Gate::denies('update-post', $post)) {
            abort(403);
        }

        // Method 3: authorize() - throws exception
        Gate::authorize('update-post', $post);
        // If we reach here, user is authorized

        $post->update($request->validated());
        return redirect()->route('posts.show', $post);
    }
}
```

### Via User Model

```php
if ($request->user()->can('update-post', $post)) {
    // User can update
}

if ($request->user()->cannot('update-post', $post)) {
    abort(403);
}
```

### In Blade Templates

```blade
@can('update-post', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan

@cannot('delete-post', $post)
    <p>You cannot delete this post.</p>
@endcannot

@canany(['update-post', 'delete-post'], $post)
    <div class="admin-actions">
        @can('update-post', $post)
            <a href="{{ route('posts.edit', $post) }}">Edit</a>
        @endcan
    </div>
@endcanany
```

## Gate Responses

Return detailed responses instead of just true/false:

```php
use Illuminate\Auth\Access\Response;

Gate::define('edit-settings', function (User $user) {
    if ($user->is_admin) {
        return Response::allow();
    }

    return Response::deny('You must be an administrator.');
});

// Using the response
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // Can edit
}

if ($response->denied()) {
    echo $response->message();  // "You must be an administrator."
}
```

## Before and After Hooks

```php
// Runs before all gate checks
Gate::before(function (User $user, string $ability) {
    // Super admins can do anything
    if ($user->isSuperAdmin()) {
        return true;
    }

    // Return null to let the normal gate check run
    return null;
});

// Runs after all gate checks
Gate::after(function (User $user, string $ability, bool|null $result) {
    // Log the authorization attempt
    Log::info("Gate check: {$ability}", [
        'user' => $user->id,
        'result' => $result,
    ]);
});
```

## Inline Authorization

```php
// Quick inline check
Gate::allowIf(fn (User $user) => $user->is_admin);
Gate::denyIf(fn (User $user) => $user->banned);

// With exception
try {
    Gate::allowIf(fn (User $user) => $user->is_admin);
} catch (AuthorizationException $e) {
    // Handle denial
}
```

## Authorizing Resource Controllers

```php
class PostController extends Controller
{
    public function __construct()
    {
        // Authorize based on controller method
        $this->authorizeResource(Post::class, 'post');
    }

    // Methods map to abilities:
    // index → viewAny
    // show → view
    // create → create
    // store → create
    // edit → update
    // update → update
    // destroy → delete
}
```

## Multiple Parameters

```php
Gate::define('move-post', function (User $user, Post $post, Category $category) {
    return $user->id === $post->user_id
        && $category->accepts_posts;
});

// Usage
Gate::allows('move-post', [$post, $category]);
```

## Guest Users

Handle unauthenticated users:

```php
// Make user parameter nullable
Gate::define('view-post', function (?User $user, Post $post) {
    if ($post->is_public) {
        return true;  // Anyone can view public posts
    }

    return $user?->id === $post->user_id;
});
```

## Resources

- [Authorization - Gates](https://laravel.com/docs/12.x/authorization#gates) — Official documentation on Gates

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*