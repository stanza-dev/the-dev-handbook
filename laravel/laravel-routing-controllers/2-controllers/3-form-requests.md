---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-form-requests"
---

# Form Request Validation

Form requests are custom request classes that contain validation logic. They keep your controllers clean and make validation reusable.

## Creating Form Requests

```bash
php artisan make:request StorePostRequest
```

This creates `app/Http/Requests/StorePostRequest.php`:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return true;  // or add authorization logic
    }

    /**
     * Get the validation rules that apply to the request.
     */
    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'body' => 'required|string',
            'category_id' => 'required|exists:categories,id',
        ];
    }
}
```

## Using Form Requests in Controllers

Type-hint the form request instead of `Request`:

```php
use App\Http\Requests\StorePostRequest;

class PostController extends Controller
{
    public function store(StorePostRequest $request)
    {
        // Validation happens automatically!
        // If validation fails, user is redirected with errors

        // Access validated data
        $validated = $request->validated();

        $post = Post::create($validated);

        return redirect()->route('posts.show', $post);
    }
}
```

## Authorization in Form Requests

```php
public function authorize(): bool
{
    // Check if user owns the resource
    $post = $this->route('post');  // Get route model binding

    return $post && $this->user()->id === $post->user_id;
}

// Or use policies
public function authorize(): bool
{
    return $this->user()->can('create', Post::class);
}
```

If authorization fails, a 403 response is returned.

## Custom Error Messages

```php
public function messages(): array
{
    return [
        'title.required' => 'Please enter a post title.',
        'title.max' => 'The title cannot exceed 255 characters.',
        'body.required' => 'The post body is required.',
        'category_id.exists' => 'Please select a valid category.',
    ];
}
```

## Custom Attribute Names

```php
public function attributes(): array
{
    return [
        'category_id' => 'category',
        'user_id' => 'author',
    ];
}
// Instead of "The category_id field is required."
// Shows: "The category field is required."
```

## Preparing Input for Validation

Modify input before validation:

```php
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->title),
        'user_id' => $this->user()->id,
    ]);
}
```

## Accessing Validated Data

```php
public function store(StorePostRequest $request)
{
    // All validated data
    $validated = $request->validated();

    // Only specific keys
    $validated = $request->safe()->only(['title', 'body']);

    // All except specific keys
    $validated = $request->safe()->except(['category_id']);

    // Merge additional data
    $validated = $request->safe()->merge(['user_id' => auth()->id()]);
}
```

## Conditional Validation Rules

```php
public function rules(): array
{
    return [
        'email' => 'required|email',
        'role' => 'required|string',
        // Only validate password for new users
        'password' => $this->isMethod('post')
            ? 'required|string|min:8'
            : 'nullable|string|min:8',
    ];
}

// Or use the sometimes method
protected function withValidator($validator)
{
    $validator->sometimes('reason', 'required', function ($input) {
        return $input->role === 'admin';
    });
}
```

## After Validation Hook

```php
protected function passedValidation(): void
{
    // Called after validation passes
    // Good for cleaning up or transforming data

    $this->replace([
        'title' => strip_tags($this->title),
    ]);
}
```

## Practical Example: UpdatePostRequest

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class UpdatePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        $post = $this->route('post');

        return $this->user()->can('update', $post);
    }

    public function rules(): array
    {
        return [
            'title' => [
                'required',
                'string',
                'max:255',
                // Unique except for this post
                Rule::unique('posts')->ignore($this->route('post')),
            ],
            'body' => 'required|string|min:100',
            'category_id' => 'required|exists:categories,id',
            'tags' => 'nullable|array',
            'tags.*' => 'exists:tags,id',
            'published_at' => 'nullable|date|after:now',
        ];
    }

    public function messages(): array
    {
        return [
            'body.min' => 'Posts must be at least 100 characters long.',
            'tags.*.exists' => 'One or more selected tags are invalid.',
        ];
    }
}
```

## Resources

- [Form Request Validation](https://laravel.com/docs/12.x/validation#form-request-validation) — Official documentation on form request validation

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*