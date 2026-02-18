---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-query-builder"
---

# Advanced Query Building

Eloquent is built on Laravel's Query Builder, giving you access to powerful query capabilities while working with models.

## Where Clauses

### Basic Where

```php
$posts = Post::where('status', 'published')->get();
$posts = Post::where('status', '=', 'published')->get();  // Same

// Different operators
$posts = Post::where('views', '>', 100)->get();
$posts = Post::where('views', '>=', 100)->get();
$posts = Post::where('views', '<>', 0)->get();  // Not equal
$posts = Post::where('title', 'like', '%Laravel%')->get();
```

### Multiple Conditions

```php
// AND conditions
$posts = Post::where('status', 'published')
    ->where('category_id', 1)
    ->get();

// Or as array
$posts = Post::where([
    ['status', 'published'],
    ['category_id', 1],
])->get();

// OR conditions
$posts = Post::where('status', 'published')
    ->orWhere('featured', true)
    ->get();

// Grouped conditions (parentheses)
$posts = Post::where('status', 'published')
    ->where(function ($query) {
        $query->where('views', '>', 1000)
              ->orWhere('featured', true);
    })
    ->get();
// WHERE status = 'published' AND (views > 1000 OR featured = true)
```

### Advanced Where Clauses

```php
// whereIn / whereNotIn
$posts = Post::whereIn('category_id', [1, 2, 3])->get();
$posts = Post::whereNotIn('status', ['draft', 'archived'])->get();

// whereBetween
$posts = Post::whereBetween('views', [100, 1000])->get();
$posts = Post::whereNotBetween('created_at', [$start, $end])->get();

// whereNull / whereNotNull
$posts = Post::whereNull('published_at')->get();
$posts = Post::whereNotNull('published_at')->get();

// whereDate / whereMonth / whereYear
$posts = Post::whereDate('created_at', '2024-01-15')->get();
$posts = Post::whereMonth('created_at', 12)->get();
$posts = Post::whereYear('created_at', 2024)->get();

// whereColumn (compare two columns)
$posts = Post::whereColumn('created_at', 'updated_at')->get();
$posts = Post::whereColumn('views', '>', 'comments_count')->get();
```

## Ordering and Pagination

```php
// Order by
$posts = Post::orderBy('created_at', 'desc')->get();
$posts = Post::latest()->get();  // Same as orderBy('created_at', 'desc')
$posts = Post::oldest()->get();  // orderBy('created_at', 'asc')
$posts = Post::inRandomOrder()->get();

// Multiple ordering
$posts = Post::orderBy('featured', 'desc')
    ->orderBy('created_at', 'desc')
    ->get();

// Pagination
$posts = Post::paginate(15);  // 15 per page
$posts = Post::simplePaginate(15);  // Simpler, no total count
$posts = Post::cursorPaginate(15);  // For large datasets

// In Blade
{{ $posts->links() }}  // Pagination links
```

## Selecting Specific Columns

```php
// Select specific columns
$posts = Post::select('title', 'slug', 'published_at')->get();

// Add columns
$posts = Post::select('title')
    ->addSelect('body')
    ->get();

// Distinct
$categories = Post::distinct()->pluck('category_id');
```

## Aggregates and Grouping

```php
// Aggregates
$count = Post::where('published', true)->count();
$maxViews = Post::max('views');
$avgRating = Post::avg('rating');
$totalViews = Post::sum('views');

// Group by
$postsByCategory = Post::select('category_id', DB::raw('COUNT(*) as count'))
    ->groupBy('category_id')
    ->get();

// Having
$popularCategories = Post::select('category_id', DB::raw('COUNT(*) as count'))
    ->groupBy('category_id')
    ->having('count', '>', 10)
    ->get();
```

## Joins

```php
// Inner join
$posts = Post::join('users', 'posts.user_id', '=', 'users.id')
    ->select('posts.*', 'users.name as author_name')
    ->get();

// Left join
$posts = Post::leftJoin('comments', 'posts.id', '=', 'comments.post_id')
    ->select('posts.*', DB::raw('COUNT(comments.id) as comments_count'))
    ->groupBy('posts.id')
    ->get();
```

## Query Scopes

Reusable query constraints:

```php
class Post extends Model
{
    /**
     * Scope: Only published posts.
     */
    public function scopePublished($query)
    {
        return $query->whereNotNull('published_at')
            ->where('published_at', '<=', now());
    }

    /**
     * Scope: Featured posts.
     */
    public function scopeFeatured($query)
    {
        return $query->where('featured', true);
    }

    /**
     * Scope: Posts by category (with parameter).
     */
    public function scopeOfCategory($query, int $categoryId)
    {
        return $query->where('category_id', $categoryId);
    }
}
```

Usage:

```php
$posts = Post::published()->get();
$posts = Post::published()->featured()->get();
$posts = Post::published()->ofCategory(1)->latest()->get();
```

## Global Scopes

Automatically applied to all queries:

```php
// In the model
protected static function booted(): void
{
    static::addGlobalScope('published', function ($query) {
        $query->whereNotNull('published_at');
    });
}

// Remove global scope for specific query
Post::withoutGlobalScope('published')->get();
```

## Raw Expressions

```php
// Raw select
$posts = Post::select(DB::raw('YEAR(created_at) as year, COUNT(*) as count'))
    ->groupBy('year')
    ->get();

// Raw where
$posts = Post::whereRaw('views > comments_count * 10')->get();

// Raw order
$posts = Post::orderByRaw('FIELD(status, "featured", "published", "draft")')->get();
```

## Resources

- [Query Builder](https://laravel.com/docs/12.x/queries) — Complete Query Builder documentation

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*