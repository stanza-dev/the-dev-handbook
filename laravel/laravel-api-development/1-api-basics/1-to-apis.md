---
source_course: "laravel-api-development"
source_lesson: "laravel-api-development-introduction-to-apis"
---

# Introduction to REST APIs in Laravel

REST (Representational State Transfer) APIs allow applications to communicate over HTTP. Laravel makes building APIs elegant and straightforward.

## What is a REST API?

REST APIs use HTTP methods to perform operations on resources:

| HTTP Method | Action | Example URL | Description |
|-------------|--------|-------------|-------------|
| GET | Read | /api/posts | List all posts |
| GET | Read | /api/posts/1 | Get post #1 |
| POST | Create | /api/posts | Create a post |
| PUT/PATCH | Update | /api/posts/1 | Update post #1 |
| DELETE | Delete | /api/posts/1 | Delete post #1 |

## API Routes in Laravel

API routes are defined in `routes/api.php`:

```php
// routes/api.php
use App\Http\Controllers\Api\PostController;

Route::get('/posts', [PostController::class, 'index']);
Route::get('/posts/{post}', [PostController::class, 'show']);
Route::post('/posts', [PostController::class, 'store']);
Route::put('/posts/{post}', [PostController::class, 'update']);
Route::delete('/posts/{post}', [PostController::class, 'destroy']);
```

Routes in `api.php` are:
- Automatically prefixed with `/api`
- Stateless (no session)
- Rate-limited by default

## Your First API Controller

```bash
php artisan make:controller Api/PostController --api
```

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class PostController extends Controller
{
    /**
     * Display a listing of posts.
     */
    public function index(): JsonResponse
    {
        $posts = Post::latest()->paginate(15);

        return response()->json($posts);
    }

    /**
     * Store a newly created post.
     */
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'body' => 'required|string',
        ]);

        $post = Post::create($validated);

        return response()->json($post, 201);
    }

    /**
     * Display the specified post.
     */
    public function show(Post $post): JsonResponse
    {
        return response()->json($post);
    }

    /**
     * Update the specified post.
     */
    public function update(Request $request, Post $post): JsonResponse
    {
        $validated = $request->validate([
            'title' => 'sometimes|required|string|max:255',
            'body' => 'sometimes|required|string',
        ]);

        $post->update($validated);

        return response()->json($post);
    }

    /**
     * Remove the specified post.
     */
    public function destroy(Post $post): JsonResponse
    {
        $post->delete();

        return response()->json(null, 204);
    }
}
```

## JSON Responses

Laravel automatically converts arrays and Eloquent models to JSON:

```php
// Arrays become JSON
return response()->json([
    'message' => 'Success',
    'data' => $post,
]);

// Models become JSON
return $post;  // Eloquent model
return Post::all();  // Collection
```

## HTTP Status Codes

```php
// Success codes
return response()->json($data, 200);  // OK (default)
return response()->json($post, 201);  // Created
return response()->json(null, 204);   // No Content (delete)

// Client error codes
return response()->json(['error' => 'Not found'], 404);
return response()->json(['error' => 'Validation failed'], 422);
return response()->json(['error' => 'Unauthorized'], 401);
return response()->json(['error' => 'Forbidden'], 403);

// Server error codes
return response()->json(['error' => 'Server error'], 500);
```

## Resource Routes

Define all CRUD routes in one line:

```php
// API resource (5 routes, no create/edit)
Route::apiResource('posts', PostController::class);

// Multiple resources
Route::apiResources([
    'posts' => PostController::class,
    'comments' => CommentController::class,
]);

// Nested resources
Route::apiResource('posts.comments', CommentController::class);
```

## Testing API Endpoints

Using cURL:

```bash
# GET all posts
curl http://localhost:8000/api/posts

# GET single post
curl http://localhost:8000/api/posts/1

# POST create post
curl -X POST http://localhost:8000/api/posts \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"title": "New Post", "body": "Content"}'

# PUT update post
curl -X PUT http://localhost:8000/api/posts/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Title"}'

# DELETE post
curl -X DELETE http://localhost:8000/api/posts/1
```

## Accept Header

Always include `Accept: application/json` to get proper error responses:

```php
// Without Accept header, validation errors may redirect
// With Accept: application/json, you get:
{
    "message": "The title field is required.",
    "errors": {
        "title": ["The title field is required."]
    }
}
```

## Resources

- [API Documentation](https://laravel.com/docs/12.x/routing#api-routes) — Laravel API routing documentation

---

> 📘 *This lesson is part of the [Laravel API Development](https://stanza.dev/courses/laravel-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*