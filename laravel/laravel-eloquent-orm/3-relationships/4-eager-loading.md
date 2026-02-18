---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-eager-loading"
---

# Eager Loading and the N+1 Problem

The N+1 problem is a common performance issue where your application makes too many database queries. Eager loading is the solution.

## The N+1 Problem

```php
// BAD: N+1 queries
$posts = Post::all();  // 1 query to get all posts

foreach ($posts as $post) {
    echo $post->user->name;  // 1 query for EACH post's user!
}

// If you have 100 posts = 101 queries total!
```

```sql
-- Query 1
SELECT * FROM posts;

-- Query 2 (for post 1)
SELECT * FROM users WHERE id = 1;

-- Query 3 (for post 2)
SELECT * FROM users WHERE id = 2;

-- ... 98 more queries!
```

## Eager Loading with with()

```php
// GOOD: Only 2 queries
$posts = Post::with('user')->get();

foreach ($posts as $post) {
    echo $post->user->name;  // No additional query!
}
```

```sql
-- Query 1
SELECT * FROM posts;

-- Query 2
SELECT * FROM users WHERE id IN (1, 2, 3, ...);  -- All users at once!
```

## Loading Multiple Relationships

```php
// Load multiple relationships
$posts = Post::with(['user', 'category', 'tags'])->get();

// Nested relationships
$posts = Post::with('user.profile')->get();  // User and their Profile

// Multiple nested
$posts = Post::with([
    'user.profile',
    'comments.author',
    'tags',
])->get();
```

## Constraining Eager Loads

```php
// Only load specific columns
$posts = Post::with('user:id,name,email')->get();

// Must include the foreign key!
$posts = Post::with('comments:id,post_id,body')->get();

// With conditions
$posts = Post::with(['comments' => function ($query) {
    $query->where('approved', true)
          ->orderBy('created_at', 'desc');
}])->get();

// Combine: load users, but only their approved comments
$users = User::with(['posts', 'posts.comments' => function ($query) {
    $query->where('approved', true);
}])->get();
```

## Lazy Eager Loading

Load relationships after the initial query:

```php
$posts = Post::all();  // Already loaded without relationships

// Realize you need users
if ($someCondition) {
    $posts->load('user');  // Load for all posts
}

// With constraints
$posts->load(['comments' => function ($query) {
    $query->where('approved', true);
}]);

// Load if not already loaded
$posts->loadMissing('user');
```

## Default Eager Loading

Always load certain relationships:

```php
class Post extends Model
{
    /**
     * The relationships that should always be loaded.
     */
    protected $with = ['user', 'category'];
}

// Now User is always loaded
$posts = Post::all();  // Includes user and category

// Override when you don't need them
$posts = Post::without('category')->get();
```

## Preventing Lazy Loading

Catch N+1 problems during development:

```php
// In AppServiceProvider boot()
use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

Now accessing a non-loaded relationship throws an exception:

```php
$post = Post::first();  // No eager loading
$post->user;  // Throws LazyLoadingViolationException in dev!
```

## Counting Related Records

```php
// Count without loading all data
$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;  // Number, not Collection
}

// Multiple counts
$users = User::withCount(['posts', 'comments'])->get();
// $user->posts_count, $user->comments_count

// Count with constraints
$posts = Post::withCount(['comments' => function ($query) {
    $query->where('approved', true);
}])->get();
```

## Aggregate Functions

```php
// Sum
$posts = Post::withSum('comments', 'upvotes')->get();
// $post->comments_sum_upvotes

// Average
$posts = Post::withAvg('ratings', 'score')->get();
// $post->ratings_avg_score

// Min/Max
$posts = Post::withMin('comments', 'created_at')->get();
$posts = Post::withMax('comments', 'created_at')->get();

// Check existence efficiently
$posts = Post::withExists('comments')->get();
// $post->comments_exists (boolean)
```

## Best Practices

```php
// ✅ Always eager load when iterating
$posts = Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name;
}

// ✅ Be specific about what you need
$posts = Post::with('user:id,name')->get();

// ✅ Use withCount for numbers
$posts = Post::withCount('comments')->get();

// ❌ Don't load everything blindly
$posts = Post::with(['user', 'comments', 'tags', 'category', ...])->get();

// ❌ Don't load what you don't use
if ($showAuthor) {
    $posts->load('user');  // Only load when needed
}
```

## Resources

- [Eager Loading](https://laravel.com/docs/12.x/eloquent-relationships#eager-loading) — Official documentation on eager loading

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*