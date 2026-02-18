---
source_course: "laravel-api-development"
source_lesson: "laravel-api-development-api-documentation"
---

# Generating API Documentation

Good API documentation is essential for developers using your API. Learn how to generate and maintain comprehensive API docs.

## Documentation Options

| Tool | Type | Pros |
|------|------|------|
| **Scribe** | Auto-generated | Parses code, generates OpenAPI spec |
| **Laravel API Documentation Generator** | Auto-generated | Simple setup |
| **Swagger/OpenAPI** | Manual/Auto | Industry standard |
| **Postman Collections** | Manual | Interactive testing |

## Using Scribe

Scribe is the most popular Laravel API documentation generator:

```bash
composer require --dev knuckleswtf/scribe

php artisan vendor:publish --tag=scribe-config
```

### Configuration

Edit `config/scribe.php`:

```php
return [
    'title' => 'My API Documentation',
    'description' => 'API for managing posts and users',
    'base_url' => env('APP_URL'),

    'routes' => [
        [
            'match' => [
                'prefixes' => ['api/*'],
                'domains' => ['*'],
            ],
            'include' => [],
            'exclude' => [],
        ],
    ],

    'auth' => [
        'enabled' => true,
        'default' => true,
        'in' => 'bearer',
        'name' => 'Authorization',
        'use_value' => 'Bearer {token}',
        'placeholder' => '{token}',
    ],
];
```

### Documenting Endpoints

Add DocBlocks to your controllers:

```php
/**
 * @group Posts
 *
 * APIs for managing blog posts
 */
class PostController extends Controller
{
    /**
     * List all posts
     *
     * Get a paginated list of all posts.
     *
     * @queryParam page integer Page number. Example: 1
     * @queryParam per_page integer Results per page. Example: 15
     * @queryParam status string Filter by status. Example: published
     *
     * @response 200 {
     *   "data": [
     *     {
     *       "id": 1,
     *       "title": "First Post",
     *       "body": "Content here...",
     *       "created_at": "2024-01-15T10:30:00Z"
     *     }
     *   ],
     *   "meta": {
     *     "current_page": 1,
     *     "total": 50
     *   }
     * }
     */
    public function index(Request $request)
    {
        return PostResource::collection(
            Post::paginate($request->per_page ?? 15)
        );
    }

    /**
     * Create a post
     *
     * Create a new blog post.
     *
     * @authenticated
     *
     * @bodyParam title string required The post title. Example: My New Post
     * @bodyParam body string required The post content. Example: This is the post body...
     * @bodyParam category_id integer The category ID. Example: 1
     * @bodyParam tags array List of tag IDs. Example: [1, 2, 3]
     *
     * @response 201 {
     *   "data": {
     *     "id": 1,
     *     "title": "My New Post",
     *     "body": "This is the post body..."
     *   }
     * }
     *
     * @response 422 {
     *   "message": "The title field is required.",
     *   "errors": {
     *     "title": ["The title field is required."]
     *   }
     * }
     */
    public function store(StorePostRequest $request)
    {
        $post = Post::create($request->validated());
        return new PostResource($post);
    }

    /**
     * Get a post
     *
     * Retrieve a single post by ID.
     *
     * @urlParam id integer required The post ID. Example: 1
     *
     * @response 200 {
     *   "data": {
     *     "id": 1,
     *     "title": "First Post",
     *     "body": "Content here..."
     *   }
     * }
     *
     * @response 404 {
     *   "message": "Post not found"
     * }
     */
    public function show(Post $post)
    {
        return new PostResource($post);
    }
}
```

### Generate Documentation

```bash
php artisan scribe:generate
```

This creates:
- `public/docs/index.html` - Interactive documentation
- `public/docs/openapi.yaml` - OpenAPI specification

### API Response Tags

```php
/**
 * @response 200 scenario="Success" {"data": {"id": 1}}
 * @response 401 scenario="Unauthenticated" {"message": "Unauthenticated"}
 * @response 403 scenario="Forbidden" {"message": "This action is unauthorized"}
 * @response 404 scenario="Not Found" {"message": "Resource not found"}
 * @response 422 scenario="Validation Error" {"message": "...", "errors": {}}
 */
```

## OpenAPI/Swagger Specification

Scribe generates an OpenAPI spec you can use with other tools:

```yaml
# public/docs/openapi.yaml
openapi: 3.0.3
info:
  title: 'My API Documentation'
  version: '1.0.0'
paths:
  /api/posts:
    get:
      summary: 'List all posts'
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: 'Success'
```

## Postman Collections

Export to Postman:

```bash
php artisan scribe:generate --postman
```

This creates `public/docs/collection.json` you can import into Postman.

## Best Practices

```php
/**
 * @group Authentication
 *
 * APIs for user authentication
 */
class AuthController extends Controller
{
    /**
     * Login
     *
     * Authenticate a user and return an access token.
     *
     * This endpoint does not require authentication.
     *
     * @unauthenticated
     *
     * @bodyParam email string required User's email. Example: user@example.com
     * @bodyParam password string required User's password. Example: password123
     * @bodyParam device_name string required Device identifier. Example: iPhone 15
     *
     * @response 200 {
     *   "user": {
     *     "id": 1,
     *     "name": "John Doe",
     *     "email": "user@example.com"
     *   },
     *   "token": "1|abc123..."
     * }
     *
     * @response 422 {
     *   "message": "The provided credentials are incorrect.",
     *   "errors": {
     *     "email": ["The provided credentials are incorrect."]
     *   }
     * }
     */
    public function login(LoginRequest $request)
    {
        // ...
    }
}
```

## Resources

- [Scribe Documentation](https://scribe.knuckles.wtf/laravel) — Official Scribe documentation for Laravel

---

> 📘 *This lesson is part of the [Laravel API Development](https://stanza.dev/courses/laravel-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*