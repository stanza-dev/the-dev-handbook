---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-resource-controllers"
---

# Resource Controllers

Resource controllers map the typical CRUD operations to controller methods with a single line of route definition. This follows RESTful conventions.

## Creating a Resource Controller

```bash
php artisan make:controller PostController --resource
```

This generates a controller with all CRUD methods:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        //
    }

    /**
     * Show the form for creating a new resource.
     */
    public function create()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        //
    }

    /**
     * Display the specified resource.
     */
    public function show(Post $post)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     */
    public function edit(Post $post)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, Post $post)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(Post $post)
    {
        //
    }
}
```

## Registering Resource Routes

One line creates all 7 routes:

```php
Route::resource('posts', PostController::class);
```

This creates:

| Verb | URI | Action | Route Name |
|------|-----|--------|------------|
| GET | /posts | index | posts.index |
| GET | /posts/create | create | posts.create |
| POST | /posts | store | posts.store |
| GET | /posts/{post} | show | posts.show |
| GET | /posts/{post}/edit | edit | posts.edit |
| PUT/PATCH | /posts/{post} | update | posts.update |
| DELETE | /posts/{post} | destroy | posts.destroy |

## Partial Resources

Only include certain methods:

```php
// Only these methods
Route::resource('posts', PostController::class)->only([
    'index', 'show'
]);

// All except these
Route::resource('posts', PostController::class)->except([
    'create', 'store', 'edit', 'update', 'destroy'
]);
```

## API Resource Controllers

For APIs, you don't need `create` and `edit` (those return forms):

```bash
php artisan make:controller API/PostController --api
```

Register API routes:

```php
// Only 5 routes (no create/edit)
Route::apiResource('posts', PostController::class);

// Multiple API resources
Route::apiResources([
    'posts' => PostController::class,
    'comments' => CommentController::class,
]);
```

## Nested Resources

For parent-child relationships:

```php
// posts/1/comments, posts/1/comments/5
Route::resource('posts.comments', CommentController::class);
```

Controller receives both models:

```php
class CommentController extends Controller
{
    public function show(Post $post, Comment $comment)
    {
        // Both $post and $comment are injected
    }
}
```

### Shallow Nesting

Use parent ID only when necessary:

```php
Route::resource('posts.comments', CommentController::class)->shallow();
```

This creates:
- `/posts/{post}/comments` (index, create, store)
- `/comments/{comment}` (show, edit, update, destroy)

## Naming Resource Routes

Customize route names:

```php
Route::resource('posts', PostController::class)->names([
    'create' => 'posts.build',
    'store' => 'posts.save',
]);

// Or prefix all names
Route::resource('posts', PostController::class)->names('admin.posts');
// admin.posts.index, admin.posts.create, etc.
```

## Custom Route Parameters

Change the parameter name:

```php
Route::resource('users', UserController::class)->parameters([
    'users' => 'admin_user'
]);
// /users/{admin_user}
```

## Implementing a Complete Resource Controller

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use App\Http\Requests\StorePostRequest;
use App\Http\Requests\UpdatePostRequest;
use Illuminate\Http\RedirectResponse;
use Illuminate\View\View;

class PostController extends Controller
{
    public function index(): View
    {
        $posts = Post::latest()->paginate(10);

        return view('posts.index', ['posts' => $posts]);
    }

    public function create(): View
    {
        return view('posts.create');
    }

    public function store(StorePostRequest $request): RedirectResponse
    {
        $post = Post::create($request->validated());

        return redirect()->route('posts.show', $post)
            ->with('success', 'Post created successfully!');
    }

    public function show(Post $post): View
    {
        return view('posts.show', ['post' => $post]);
    }

    public function edit(Post $post): View
    {
        return view('posts.edit', ['post' => $post]);
    }

    public function update(UpdatePostRequest $request, Post $post): RedirectResponse
    {
        $post->update($request->validated());

        return redirect()->route('posts.show', $post)
            ->with('success', 'Post updated successfully!');
    }

    public function destroy(Post $post): RedirectResponse
    {
        $post->delete();

        return redirect()->route('posts.index')
            ->with('success', 'Post deleted successfully!');
    }
}
```

## Resources

- [Resource Controllers](https://laravel.com/docs/12.x/controllers#resource-controllers) — Official documentation on resource controllers

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*