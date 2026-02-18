---
source_course: "laravel-api-development"
source_lesson: "laravel-api-development-api-validation"
---

# API Validation and Error Handling

Proper validation and error handling are crucial for API reliability. Laravel provides tools to make both elegant and consistent.

## Validation in API Controllers

```php
public function store(Request $request): JsonResponse
{
    $validated = $request->validate([
        'title' => 'required|string|max:255',
        'body' => 'required|string',
        'category_id' => 'required|exists:categories,id',
        'tags' => 'nullable|array',
        'tags.*' => 'exists:tags,id',
    ]);

    $post = Post::create($validated);

    return response()->json($post, 201);
}
```

With `Accept: application/json`, validation errors return:

```json
{
    "message": "The title field is required. (and 1 more error)",
    "errors": {
        "title": ["The title field is required."],
        "body": ["The body field is required."]
    }
}
```

## Form Request Validation

```bash
php artisan make:request StorePostRequest
```

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Contracts\Validation\Validator;
use Illuminate\Http\Exceptions\HttpResponseException;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'body' => 'required|string|min:100',
            'category_id' => 'required|exists:categories,id',
            'tags' => 'nullable|array|max:5',
            'tags.*' => 'exists:tags,id',
            'published_at' => 'nullable|date|after:now',
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'Please provide a title for your post.',
            'body.min' => 'Post content must be at least 100 characters.',
            'tags.max' => 'You can only select up to 5 tags.',
        ];
    }

    /**
     * Customize the failed validation response for APIs.
     */
    protected function failedValidation(Validator $validator): void
    {
        throw new HttpResponseException(
            response()->json([
                'success' => false,
                'message' => 'Validation errors',
                'errors' => $validator->errors(),
            ], 422)
        );
    }
}
```

## Custom Error Responses

Customize error handling in `bootstrap/app.php`:

```php
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Illuminate\Validation\ValidationException;
use Illuminate\Auth\AuthenticationException;

return Application::configure(basePath: dirname(__DIR__))
    ->withExceptions(function (Exceptions $exceptions) {
        // Not Found (404)
        $exceptions->render(function (NotFoundHttpException $e, Request $request) {
            if ($request->is('api/*') || $request->wantsJson()) {
                return response()->json([
                    'success' => false,
                    'message' => 'Resource not found.',
                ], 404);
            }
        });

        // Authentication (401)
        $exceptions->render(function (AuthenticationException $e, Request $request) {
            if ($request->is('api/*') || $request->wantsJson()) {
                return response()->json([
                    'success' => false,
                    'message' => 'Unauthenticated.',
                ], 401);
            }
        });

        // Validation (422)
        $exceptions->render(function (ValidationException $e, Request $request) {
            if ($request->is('api/*') || $request->wantsJson()) {
                return response()->json([
                    'success' => false,
                    'message' => 'Validation failed.',
                    'errors' => $e->errors(),
                ], 422);
            }
        });
    })
    ->create();
```

## Consistent API Responses

Create a response helper:

```php
// app/Traits/ApiResponses.php
<?php

namespace App\Traits;

use Illuminate\Http\JsonResponse;

trait ApiResponses
{
    protected function success($data = null, string $message = 'Success', int $code = 200): JsonResponse
    {
        return response()->json([
            'success' => true,
            'message' => $message,
            'data' => $data,
        ], $code);
    }

    protected function created($data = null, string $message = 'Created successfully'): JsonResponse
    {
        return $this->success($data, $message, 201);
    }

    protected function noContent(): JsonResponse
    {
        return response()->json(null, 204);
    }

    protected function error(string $message, int $code = 400, $errors = null): JsonResponse
    {
        $response = [
            'success' => false,
            'message' => $message,
        ];

        if ($errors) {
            $response['errors'] = $errors;
        }

        return response()->json($response, $code);
    }

    protected function notFound(string $message = 'Resource not found'): JsonResponse
    {
        return $this->error($message, 404);
    }

    protected function unauthorized(string $message = 'Unauthorized'): JsonResponse
    {
        return $this->error($message, 401);
    }

    protected function forbidden(string $message = 'Forbidden'): JsonResponse
    {
        return $this->error($message, 403);
    }
}
```

Usage:

```php
class PostController extends Controller
{
    use ApiResponses;

    public function index()
    {
        $posts = Post::paginate(15);
        return $this->success(PostResource::collection($posts));
    }

    public function store(StorePostRequest $request)
    {
        $post = Post::create($request->validated());
        return $this->created(new PostResource($post));
    }

    public function show(Post $post)
    {
        return $this->success(new PostResource($post));
    }

    public function destroy(Post $post)
    {
        $post->delete();
        return $this->noContent();
    }
}
```

## Try-Catch for Edge Cases

```php
public function store(StorePostRequest $request): JsonResponse
{
    try {
        $post = DB::transaction(function () use ($request) {
            $post = Post::create($request->validated());
            $post->tags()->attach($request->tags);
            return $post;
        });

        return $this->created(new PostResource($post->load('tags')));
    } catch (\Exception $e) {
        Log::error('Post creation failed', [
            'error' => $e->getMessage(),
            'user' => auth()->id(),
        ]);

        return $this->error('Failed to create post. Please try again.', 500);
    }
}
```

## Resources

- [Validation](https://laravel.com/docs/12.x/validation) — Laravel validation documentation

---

> 📘 *This lesson is part of the [Laravel API Development](https://stanza.dev/courses/laravel-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*