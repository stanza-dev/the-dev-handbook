---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-query-optimization"
---

# Query Optimization Techniques

Optimizing database queries is crucial for application performance. Learn techniques to write efficient queries and avoid common pitfalls.

## Preventing N+1 Queries

The most common performance problem:

```php
// ❌ N+1 Problem: 101 queries for 100 posts
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->user->name;  // Query for each post!
}

// ✅ Eager loading: 2 queries total
$posts = Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name;  // No additional query
}
```

### Enable Strict Mode in Development

```php
// AppServiceProvider
public function boot(): void
{
    Model::preventLazyLoading(! app()->isProduction());
}
```

Now lazy loading throws an exception during development!

## Select Only What You Need

```php
// ❌ Selects all columns
$users = User::all();

// ✅ Select only needed columns
$users = User::select('id', 'name', 'email')->get();

// ✅ When eager loading, specify columns (include foreign keys!)
$posts = Post::with('user:id,name')->get();

// ❌ Pluck still fetches all columns first
$names = User::all()->pluck('name');

// ✅ Use pluck directly on query
$names = User::pluck('name');
```

## Use Chunking for Large Datasets

```php
// ❌ Loads all records into memory
$users = User::all();
foreach ($users as $user) {
    // Process
}

// ✅ Process in chunks
User::chunk(100, function ($users) {
    foreach ($users as $user) {
        // Process
    }
});

// ✅ Even better: lazy loading
foreach (User::lazy() as $user) {
    // Process one at a time
}

// ✅ Best for memory: cursor
foreach (User::cursor() as $user) {
    // Minimal memory usage
}
```

### When to Use Each

| Method | Memory | Use Case |
|--------|--------|----------|
| `get()` | High | Small datasets |
| `chunk()` | Medium | Updates with save() |
| `lazy()` | Low | Read-only operations |
| `cursor()` | Lowest | Very large datasets |

## Optimize Counts

```php
// ❌ Loads all models just to count
$count = Post::all()->count();

// ✅ Count at database level
$count = Post::count();

// ❌ Loads all comments to count
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->comments->count();
}

// ✅ Use withCount
$posts = Post::withCount('comments')->get();
foreach ($posts as $post) {
    echo $post->comments_count;  // Already loaded!
}
```

## Conditional Eager Loading

```php
// Load based on conditions
$posts = Post::when($includeAuthor, function ($query) {
    $query->with('user');
})->get();

// Load after the fact if needed
$posts = Post::all();
if ($showComments) {
    $posts->load('comments');  // Single query for all
}

// Don't re-load if already loaded
$posts->loadMissing('tags');
```

## Database Indexing

```php
// In migration - add indexes for frequently queried columns
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained();
    $table->string('status');
    $table->timestamp('published_at')->nullable();
    $table->timestamps();

    // Composite index for common query
    $table->index(['status', 'published_at']);
    $table->index(['user_id', 'status']);
});
```

Queries that benefit:

```php
// Uses the index
Post::where('status', 'published')
    ->where('published_at', '<=', now())
    ->get();
```

## Optimize Existence Checks

```php
// ❌ Loads the whole model
if (User::find($id)) {
    // ...
}

// ✅ Just check existence
if (User::where('id', $id)->exists()) {
    // ...
}

// ❌ Loads all related models
if ($user->posts->count() > 0) {
    // ...
}

// ✅ Check at database level
if ($user->posts()->exists()) {
    // ...
}
```

## Use Query Caching

```php
use Illuminate\Support\Facades\Cache;

// Cache expensive queries
$posts = Cache::remember('popular_posts', 3600, function () {
    return Post::withCount('comments')
        ->orderBy('comments_count', 'desc')
        ->take(10)
        ->get();
});

// Invalidate when data changes
// In PostObserver:
public function saved(Post $post): void
{
    Cache::forget('popular_posts');
}
```

## Raw Queries for Complex Operations

```php
// Sometimes raw SQL is more efficient
$results = DB::select('
    SELECT users.*, COUNT(posts.id) as posts_count
    FROM users
    LEFT JOIN posts ON users.id = posts.user_id
    WHERE users.active = 1
    GROUP BY users.id
    HAVING posts_count > 10
    ORDER BY posts_count DESC
    LIMIT 10
');

// Or use query builder raw expressions
$users = User::select('users.*')
    ->selectRaw('COUNT(posts.id) as posts_count')
    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
    ->where('users.active', true)
    ->groupBy('users.id')
    ->havingRaw('COUNT(posts.id) > 10')
    ->orderByDesc('posts_count')
    ->limit(10)
    ->get();
```

## Monitor Your Queries

```php
// Log all queries in development
DB::listen(function ($query) {
    Log::debug('Query', [
        'sql' => $query->sql,
        'bindings' => $query->bindings,
        'time' => $query->time,
    ]);
});

// Or use Laravel Debugbar / Telescope
```

## Resources

- [Database Performance](https://laravel.com/docs/12.x/eloquent#retrieving-models) — Eloquent retrieval and performance tips

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*