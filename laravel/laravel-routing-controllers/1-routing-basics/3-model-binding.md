---
source_course: "laravel-routing-controllers"
source_lesson: "laravel-routing-controllers-route-model-binding"
---

# Route Model Binding Deep Dive

Route model binding automatically injects model instances into your routes. Instead of manually querying the database, Laravel does it for you.

## Implicit Binding

Laravel automatically resolves Eloquent models by matching variable names to route segments:

```php
// Route definition
Route::get('/users/{user}', function (User $user) {
    return $user;
});

// When visiting /users/1:
// 1. Laravel sees {user} in the route
// 2. Sees User $user type-hint in the function
// 3. Automatically does User::findOrFail(1)
// 4. Injects the User instance into your function
```

### In Controllers

```php
// routes/web.php
Route::get('/posts/{post}', [PostController::class, 'show']);

// app/Http/Controllers/PostController.php
class PostController extends Controller
{
    public function show(Post $post)  // Automatically resolved!
    {
        return view('posts.show', ['post' => $post]);
    }
}
```

### Customizing the Key

By default, models are found by their primary key. Use a different column:

```php
// In the route
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
// /posts/my-first-post → finds by slug column

// Or in the model
class Post extends Model
{
    /**
     * Get the route key for the model.
     */
    public function getRouteKeyName(): string
    {
        return 'slug';
    }
}
// Now {post} always resolves by slug
```

## Scoped Bindings

Ensure child models belong to parent models:

```php
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
});

// Without scoping: /users/1/posts/5 would return post 5 even if it belongs to user 2
// With scoping: Returns 404 if post doesn't belong to user
```

Enable scoping:

```php
// Method 1: Use scopeBindings()
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();

// Method 2: In route group
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', ...);
});
```

This requires the `posts()` relationship on the User model:

```php
class User extends Model
{
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

## Explicit Binding

Customize how models are resolved in `AppServiceProvider`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

public function boot(): void
{
    // Custom resolution logic
    Route::bind('user', function (string $value) {
        return User::where('name', $value)->firstOrFail();
    });
}

// Now {user} resolves by name instead of ID
```

## Custom Resolution Logic in Models

Override how a model resolves route bindings:

```php
class User extends Model
{
    /**
     * Retrieve the model for a bound value.
     */
    public function resolveRouteBinding($value, $field = null): ?Model
    {
        // Custom logic: include soft-deleted for admins
        return $this->where($field ?? 'id', $value)
                    ->when(auth()->user()?->isAdmin(), fn ($q) => $q->withTrashed())
                    ->firstOrFail();
    }
}
```

## Handling Missing Models

By default, missing models return 404. Customize this:

```php
Route::get('/users/{user}', function (User $user) {
    return $user;
})->missing(function (Request $request) {
    return redirect()->route('users.index')
        ->with('error', 'User not found');
});
```

## Soft Deleted Models

Include soft-deleted models:

```php
Route::get('/users/{user}', function (User $user) {
    return $user;
})->withTrashed();  // Include soft-deleted users
```

## Enum Binding

PHP 8.1+ backed enums work with route binding:

```php
// Define an enum
enum Category: string
{
    case Fruits = 'fruits';
    case Vegetables = 'vegetables';
}

// Use in route
Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
// /categories/fruits → 'fruits'
// /categories/invalid → 404
```

## Best Practices

```php
// ✅ Good: Use model binding
Route::get('/users/{user}', function (User $user) {
    return $user;
});

// ❌ Avoid: Manual lookup
Route::get('/users/{id}', function ($id) {
    $user = User::findOrFail($id);  // Unnecessary
    return $user;
});

// ✅ Good: Scope nested resources
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

## Resources

- [Route Model Binding](https://laravel.com/docs/12.x/routing#route-model-binding) — Official documentation on route model binding

---

> 📘 *This lesson is part of the [Laravel Routing & Controllers](https://stanza.dev/courses/laravel-routing-controllers) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*