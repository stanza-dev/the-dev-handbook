---
source_course: "laravel-authentication"
source_lesson: "laravel-authentication-policies"
---

# Authorization with Policies

Policies are classes that organize authorization logic around a particular model. They're more structured than gates and better for complex authorization.

## Creating Policies

```bash
# Create a policy for the Post model
php artisan make:policy PostPolicy --model=Post
```

This creates `app/Policies/PostPolicy.php`:

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine if the user can view any posts.
     */
    public function viewAny(User $user): bool
    {
        return true;  // Anyone logged in can view posts list
    }

    /**
     * Determine if the user can view the post.
     */
    public function view(User $user, Post $post): bool
    {
        // Public posts or own posts
        return $post->is_public || $user->id === $post->user_id;
    }

    /**
     * Determine if the user can create posts.
     */
    public function create(User $user): bool
    {
        return $user->email_verified_at !== null;
    }

    /**
     * Determine if the user can update the post.
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    /**
     * Determine if the user can delete the post.
     */
    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    /**
     * Determine if the user can restore the post.
     */
    public function restore(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    /**
     * Determine if the user can permanently delete the post.
     */
    public function forceDelete(User $user, Post $post): bool
    {
        return $user->is_admin;
    }
}
```

## Registering Policies

Laravel auto-discovers policies by convention:

```
Model: App\Models\Post
Policy: App\Policies\PostPolicy
```

For custom mapping, register in `AppServiceProvider`:

```php
use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::policy(Post::class, PostPolicy::class);
}
```

## Using Policies

### In Controllers

```php
class PostController extends Controller
{
    public function index()
    {
        $this->authorize('viewAny', Post::class);

        return view('posts.index', ['posts' => Post::all()]);
    }

    public function show(Post $post)
    {
        $this->authorize('view', $post);

        return view('posts.show', compact('post'));
    }

    public function edit(Post $post)
    {
        $this->authorize('update', $post);

        return view('posts.edit', compact('post'));
    }

    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);

        $post->update($request->validated());

        return redirect()->route('posts.show', $post);
    }

    public function destroy(Post $post)
    {
        $this->authorize('delete', $post);

        $post->delete();

        return redirect()->route('posts.index');
    }
}
```

### Via User Model

```php
if ($user->can('update', $post)) {
    // User can update
}

if ($user->cannot('delete', $post)) {
    // User cannot delete
}

// For actions without models
if ($user->can('create', Post::class)) {
    // User can create posts
}
```

### Via Gate Facade

```php
if (Gate::allows('update', $post)) {
    // Allowed
}

Gate::authorize('delete', $post);
```

### In Blade

```blade
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan

@can('create', App\Models\Post::class)
    <a href="{{ route('posts.create') }}">Create Post</a>
@endcan

@cannot('delete', $post)
    <p>You cannot delete this post.</p>
@endcannot
```

### Via Middleware

```php
Route::put('/posts/{post}', [PostController::class, 'update'])
    ->middleware('can:update,post');

// For creation (no model)
Route::post('/posts', [PostController::class, 'store'])
    ->middleware('can:create,App\Models\Post');
```

## Policy Methods Reference

| Method | Resource Action | Model Parameter |
|--------|-----------------|------------------|
| viewAny | index | No |
| view | show | Yes |
| create | create, store | No |
| update | edit, update | Yes |
| delete | destroy | Yes |
| restore | restore | Yes |
| forceDelete | forceDelete | Yes |

## Before Method

Run before any other policy methods:

```php
public function before(User $user, string $ability): bool|null
{
    // Admins can do everything
    if ($user->is_admin) {
        return true;
    }

    // Return null to let normal authorization run
    return null;
}
```

## Guest Users

```php
// Make user nullable to allow guests
public function view(?User $user, Post $post): bool
{
    if ($post->is_public) {
        return true;  // Guests can view public posts
    }

    return $user?->id === $post->user_id;
}
```

## Policy Responses

```php
use Illuminate\Auth\Access\Response;

public function update(User $user, Post $post): Response
{
    if ($user->id !== $post->user_id) {
        return Response::deny('You do not own this post.');
    }

    if ($post->is_locked) {
        return Response::deny('This post is locked and cannot be edited.');
    }

    return Response::allow();
}
```

## Resources

- [Authorization - Policies](https://laravel.com/docs/12.x/authorization#creating-policies) — Official documentation on Policies

---

> 📘 *This lesson is part of the [Laravel Authentication & Authorization](https://stanza.dev/courses/laravel-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*