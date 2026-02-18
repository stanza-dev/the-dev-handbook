---
source_course: "laravel-api-development"
source_lesson: "laravel-api-development-api-resources"
---

# API Resources: Transforming Data

API Resources provide a transformation layer between your Eloquent models and the JSON responses. They give you full control over how your data is structured.

## Why Use Resources?

```php
// Without resources - exposes all model data
return User::find(1);
// {"id":1,"name":"John","email":"john@example.com","password":"$2y$...","remember_token":"abc..."}

// With resources - control what's exposed
return new UserResource(User::find(1));
// {"id":1,"name":"John","email":"john@example.com"}
```

## Creating Resources

```bash
# Single resource
php artisan make:resource UserResource

# Collection resource (for arrays)
php artisan make:resource UserCollection
```

## Basic Resource

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at->toISOString(),
        ];
    }
}
```

Usage:

```php
use App\Http\Resources\UserResource;

// Single resource
public function show(User $user)
{
    return new UserResource($user);
}

// Collection
public function index()
{
    return UserResource::collection(User::all());
}

// With pagination
public function index()
{
    return UserResource::collection(User::paginate(15));
}
```

## Resource with Relationships

```php
class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
            'author' => new UserResource($this->user),
            'comments' => CommentResource::collection($this->comments),
            'tags' => TagResource::collection($this->tags),
        ];
    }
}
```

## Conditional Attributes

```php
class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,

            // Only include if loaded (prevents N+1)
            'author' => new UserResource($this->whenLoaded('user')),
            'comments' => CommentResource::collection($this->whenLoaded('comments')),

            // Conditional based on value
            'secret' => $this->when($request->user()?->isAdmin(), $this->secret),

            // Conditional based on request
            'body' => $this->when($request->has('include_body'), $this->body),

            // Merge conditionally
            $this->mergeWhen($request->user()?->isAdmin(), [
                'internal_id' => $this->internal_id,
                'notes' => $this->admin_notes,
            ]),
        ];
    }
}
```

## Resource Collections

For customizing collection responses:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class PostCollection extends ResourceCollection
{
    /**
     * The resource that this resource collects.
     */
    public $collects = PostResource::class;

    /**
     * Transform the resource collection into an array.
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'meta' => [
                'total' => $this->collection->count(),
                'has_more' => $this->hasMorePages(),
            ],
        ];
    }
}
```

## Adding Meta Data

```php
class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
        ];
    }

    public function with(Request $request): array
    {
        return [
            'meta' => [
                'version' => '1.0',
                'generated_at' => now()->toISOString(),
            ],
        ];
    }
}
```

Response:

```json
{
    "data": {
        "id": 1,
        "title": "My Post"
    },
    "meta": {
        "version": "1.0",
        "generated_at": "2024-01-15T10:30:00.000Z"
    }
}
```

## Customizing the Response

```php
class PostResource extends JsonResource
{
    public function withResponse(Request $request, JsonResponse $response): void
    {
        $response->header('X-Resource-Version', '1.0');

        if ($this->is_premium) {
            $response->header('X-Premium-Content', 'true');
        }
    }
}
```

## Wrapping Data

By default, resources wrap data in a `data` key:

```json
{
    "data": {
        "id": 1,
        "name": "John"
    }
}
```

Disable wrapping:

```php
// In AppServiceProvider boot()
JsonResource::withoutWrapping();

// Or per resource
class UserResource extends JsonResource
{
    public static $wrap = null;  // Disable
    public static $wrap = 'user';  // Custom key
}
```

## Complete Example

```php
class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'slug' => $this->slug,
            'excerpt' => Str::limit($this->body, 150),
            'body' => $this->when($request->routeIs('posts.show'), $this->body),
            'featured_image' => $this->featured_image
                ? asset('storage/' . $this->featured_image)
                : null,
            'is_published' => $this->published_at !== null,
            'published_at' => $this->published_at?->toISOString(),
            'author' => new UserResource($this->whenLoaded('user')),
            'category' => new CategoryResource($this->whenLoaded('category')),
            'tags' => TagResource::collection($this->whenLoaded('tags')),
            'comments_count' => $this->whenCounted('comments'),
            'links' => [
                'self' => route('api.posts.show', $this),
            ],
        ];
    }
}
```

## Resources

- [Eloquent API Resources](https://laravel.com/docs/12.x/eloquent-resources) — Official API Resources documentation

---

> 📘 *This lesson is part of the [Laravel API Development](https://stanza.dev/courses/laravel-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*