---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-many-to-many-relationships"
---

# Many-to-Many Relationships

Many-to-many relationships connect records where each side can have multiple related records. For example, Posts can have many Tags, and Tags can belong to many Posts.

## The Pivot Table

Many-to-many requires an intermediate (pivot) table:

```
posts                post_tag (pivot)           tags
┌────┬──────────┐    ┌─────────┬────────┐       ┌────┬─────────┐
│ id │ title    │    │ post_id │ tag_id │       │ id │ name    │
├────┼──────────┤    ├─────────┼────────┤       ├────┼─────────┤
│ 1  │ "Post A" │◄──│ 1       │ 1      │──────►│ 1  │ "PHP"   │
│    │          │◄──│ 1       │ 2      │──┐    │ 2  │ "Laravel│
│ 2  │ "Post B" │◄──│ 2       │ 2      │──┴───►│    │         │
└────┴──────────┘    └─────────┴────────┘       └────┴─────────┘

Post A has tags: PHP, Laravel
Post B has tags: Laravel
PHP tag is on: Post A
Laravel tag is on: Post A, Post B
```

## Creating the Pivot Table

```bash
php artisan make:migration create_post_tag_table
```

```php
// Convention: singular table names in alphabetical order
Schema::create('post_tag', function (Blueprint $table) {
    $table->foreignId('post_id')->constrained()->cascadeOnDelete();
    $table->foreignId('tag_id')->constrained()->cascadeOnDelete();
    $table->primary(['post_id', 'tag_id']);
    $table->timestamps();  // Optional but useful
});
```

## Defining the Relationship

### On Post Model

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Post extends Model
{
    /**
     * Get the tags for the post.
     */
    public function tags(): BelongsToMany
    {
        return $this->belongsToMany(Tag::class);
    }
}
```

### On Tag Model

```php
class Tag extends Model
{
    /**
     * Get the posts with this tag.
     */
    public function posts(): BelongsToMany
    {
        return $this->belongsToMany(Post::class);
    }
}
```

## Accessing Related Records

```php
// Get all tags for a post
$post = Post::find(1);
foreach ($post->tags as $tag) {
    echo $tag->name;  // "PHP", "Laravel"
}

// Get all posts with a tag
$tag = Tag::where('name', 'Laravel')->first();
foreach ($tag->posts as $post) {
    echo $post->title;
}
```

## Attaching and Detaching

```php
$post = Post::find(1);

// Attach single tag
$post->tags()->attach($tagId);

// Attach multiple tags
$post->tags()->attach([1, 2, 3]);

// Attach with pivot data
$post->tags()->attach($tagId, ['added_by' => auth()->id()]);

// Attach multiple with pivot data
$post->tags()->attach([
    1 => ['added_by' => auth()->id()],
    2 => ['added_by' => auth()->id()],
]);

// Detach
$post->tags()->detach($tagId);       // Single
$post->tags()->detach([1, 2, 3]);    // Multiple
$post->tags()->detach();             // All tags
```

## Syncing

Replace all existing relationships:

```php
// Replace all tags with these
$post->tags()->sync([1, 2, 3]);

// With pivot data
$post->tags()->sync([
    1 => ['added_by' => auth()->id()],
    2,  // No pivot data
    3 => ['added_by' => auth()->id()],
]);

// Sync without detaching existing
$post->tags()->syncWithoutDetaching([1, 2, 3]);
```

## Toggle

Attach if not attached, detach if attached:

```php
$post->tags()->toggle([1, 2, 3]);
```

## Pivot Table Data

Access pivot table columns:

```php
// First, tell Eloquent about extra columns
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class)
        ->withPivot('added_by', 'approved_at')
        ->withTimestamps();  // Include created_at, updated_at
}

// Access pivot data
$post = Post::find(1);
foreach ($post->tags as $tag) {
    echo $tag->pivot->created_at;
    echo $tag->pivot->added_by;
}
```

## Customizing the Pivot Table

```php
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class)
        ->as('tagging')  // Rename pivot accessor
        ->withPivot('added_by')
        ->withTimestamps();
}

// Access
foreach ($post->tags as $tag) {
    echo $tag->tagging->created_at;  // Instead of $tag->pivot
}
```

## Filtering by Pivot Values

```php
// Only approved tags
$approvedTags = $post->tags()->wherePivot('approved', true)->get();

// Tags added this month
$recentTags = $post->tags()
    ->wherePivot('created_at', '>=', now()->subMonth())
    ->get();

// Using wherePivotIn
$tags = $post->tags()->wherePivotIn('added_by', [1, 2, 3])->get();
```

## Ordering by Pivot

```php
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class)
        ->withTimestamps()
        ->orderByPivot('created_at', 'desc');
}
```

## Pivot Models (Custom Pivot Classes)

```php
use Illuminate\Database\Eloquent\Relations\Pivot;

class PostTag extends Pivot
{
    protected $casts = [
        'approved_at' => 'datetime',
    ];

    public function approvedBy(): BelongsTo
    {
        return $this->belongsTo(User::class, 'approved_by');
    }
}

// In Post model
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class)
        ->using(PostTag::class)
        ->withPivot('approved_by', 'approved_at');
}
```

## Resources

- [Many-to-Many Relationships](https://laravel.com/docs/12.x/eloquent-relationships#many-to-many) — Official documentation on many-to-many relationships

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*