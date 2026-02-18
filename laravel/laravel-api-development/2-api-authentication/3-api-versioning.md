---
source_course: "laravel-api-development"
source_lesson: "laravel-api-development-api-versioning"
---

# API Versioning

API versioning allows you to make breaking changes without affecting existing API consumers. There are several strategies—let's explore the most common approaches.

## URL Versioning (Most Common)

Version in the URL path:

```
/api/v1/posts
/api/v2/posts
```

### Route Organization

```php
// routes/api.php

// Version 1
Route::prefix('v1')->group(function () {
    Route::apiResource('posts', App\Http\Controllers\Api\V1\PostController::class);
    Route::apiResource('users', App\Http\Controllers\Api\V1\UserController::class);
});

// Version 2
Route::prefix('v2')->group(function () {
    Route::apiResource('posts', App\Http\Controllers\Api\V2\PostController::class);
    Route::apiResource('users', App\Http\Controllers\Api\V2\UserController::class);
});
```

### Controller Organization

```
app/Http/Controllers/Api/
├── V1/
│   ├── PostController.php
│   └── UserController.php
├── V2/
│   ├── PostController.php
│   └── UserController.php
└── Controller.php
```

### Version-Specific Resources

```
app/Http/Resources/
├── V1/
│   ├── PostResource.php
│   └── UserResource.php
└── V2/
    ├── PostResource.php
    └── UserResource.php
```

## Header Versioning

Version via Accept header:

```
Accept: application/vnd.myapi.v1+json
Accept: application/vnd.myapi.v2+json
```

### Middleware Implementation

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class ApiVersion
{
    public function handle(Request $request, Closure $next, string $version)
    {
        $accept = $request->header('Accept', '');

        // Check for versioned media type
        if (preg_match('/application\/vnd\.myapi\.v(\d+)\+json/', $accept, $matches)) {
            $requestedVersion = $matches[1];

            if ($requestedVersion !== $version) {
                return response()->json([
                    'error' => 'API version mismatch',
                    'message' => "This endpoint requires v{$version}",
                ], 406);
            }
        }

        return $next($request);
    }
}
```

## Query Parameter Versioning

```
/api/posts?version=1
/api/posts?version=2
```

Less common, not recommended for production APIs.

## Practical URL Versioning Setup

```php
// routes/api/v1.php
<?php

use App\Http\Controllers\Api\V1\PostController;
use App\Http\Controllers\Api\V1\UserController;
use App\Http\Controllers\Api\V1\CommentController;

Route::apiResource('posts', PostController::class);
Route::apiResource('users', UserController::class);
Route::apiResource('posts.comments', CommentController::class)->shallow();
```

```php
// routes/api/v2.php
<?php

use App\Http\Controllers\Api\V2\PostController;
use App\Http\Controllers\Api\V2\UserController;

// V2 changes: posts have different response format
Route::apiResource('posts', PostController::class);
Route::apiResource('users', UserController::class);

// New endpoint in V2
Route::get('posts/{post}/related', [PostController::class, 'related']);
```

```php
// routes/api.php
<?php

// Include version-specific routes
Route::prefix('v1')
    ->middleware(['auth:sanctum', 'throttle:api'])
    ->group(base_path('routes/api/v1.php'));

Route::prefix('v2')
    ->middleware(['auth:sanctum', 'throttle:api'])
    ->group(base_path('routes/api/v2.php'));

// Default to latest stable version
Route::middleware(['auth:sanctum', 'throttle:api'])
    ->group(base_path('routes/api/v2.php'));
```

## Version-Specific Controllers

```php
// app/Http/Controllers/Api/V1/PostController.php
namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Resources\V1\PostResource;
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        return PostResource::collection(Post::paginate(15));
    }

    public function show(Post $post)
    {
        return new PostResource($post);
    }
}
```

```php
// app/Http/Controllers/Api/V2/PostController.php
namespace App\Http\Controllers\Api\V2;

use App\Http\Controllers\Controller;
use App\Http\Resources\V2\PostResource;
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        // V2: Always include author and comments count
        return PostResource::collection(
            Post::with('user')->withCount('comments')->paginate(15)
        );
    }

    public function show(Post $post)
    {
        return new PostResource($post->load('user', 'tags', 'comments.user'));
    }

    // New in V2
    public function related(Post $post)
    {
        $related = Post::where('category_id', $post->category_id)
            ->where('id', '!=', $post->id)
            ->limit(5)
            ->get();

        return PostResource::collection($related);
    }
}
```

## Best Practices

1. **Support multiple versions** during transition periods
2. **Document deprecation timelines** clearly
3. **Version early** - it's easier to add versioning from the start
4. **Don't over-version** - minor changes don't need new versions
5. **Use semantic versioning** for breaking changes only

```php
// Deprecation header
public function index()
{
    return PostResource::collection(Post::all())
        ->response()
        ->header('X-API-Deprecation-Date', '2025-01-01')
        ->header('X-API-Deprecation-Info', 'Use /api/v2/posts instead');
}
```

## Resources

- [API Best Practices](https://laravel.com/docs/12.x/routing#route-groups) — Route groups for API versioning

---

> 📘 *This lesson is part of the [Laravel API Development](https://stanza.dev/courses/laravel-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*