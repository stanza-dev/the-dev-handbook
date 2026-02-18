---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-one-to-many-relationships"
---

# One-to-Many Relationships

The most common relationship type. One parent record has many child records. For example, a User has many Posts, or a Post has many Comments.

## Defining One-to-Many

### hasMany (Parent Side)

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get the user's posts.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

### belongsTo (Child Side)

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    /**
     * Get the user who wrote the post.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

## Database Structure

```
users                          posts
┌────┬──────────┐              ┌────┬─────────┬─────────────────┐
│ id │ name     │              │ id │ user_id │ title           │
├────┼──────────┤              ├────┼─────────┼─────────────────┤
│ 1  │ "John"   │◄────────────│ 1  │ 1       │ "First Post"    │
│    │          │◄────────────│ 2  │ 1       │ "Second Post"   │
│ 2  │ "Jane"   │◄────────────│ 3  │ 2       │ "Jane's Post"   │
└────┴──────────┘              └────┴─────────┴─────────────────┘
                                    One user has MANY posts
```

## Accessing Related Records

```php
// Get all posts for a user (returns Collection)
$user = User::find(1);
$posts = $user->posts;  // Collection of Post models

foreach ($user->posts as $post) {
    echo $post->title;
}

// Query the relationship
$publishedPosts = $user->posts()
    ->where('published', true)
    ->orderBy('created_at', 'desc')
    ->get();

// Count without loading
$postCount = $user->posts()->count();

// Get parent from child
$post = Post::find(1);
$author = $post->user;  // Returns User model

echo $post->user->name;  // "John"
```

## Creating Related Records

```php
$user = User::find(1);

// Method 1: save() - single record
$post = new Post(['title' => 'My Post', 'body' => '...']);
$user->posts()->save($post);

// Method 2: create() - single record
$post = $user->posts()->create([
    'title' => 'My Post',
    'body' => '...',
]);

// Method 3: saveMany() - multiple records
$user->posts()->saveMany([
    new Post(['title' => 'Post 1']),
    new Post(['title' => 'Post 2']),
]);

// Method 4: createMany() - multiple records
$user->posts()->createMany([
    ['title' => 'Post 1', 'body' => '...'],
    ['title' => 'Post 2', 'body' => '...'],
]);
```

## Updating Parent from Child

```php
$post = Post::find(1);

// Associate a different user
$newUser = User::find(2);
$post->user()->associate($newUser);
$post->save();

// Disassociate (set user_id to null)
$post->user()->dissociate();
$post->save();
```

## Relationship Methods vs Properties

```php
// Property access - returns cached Collection or null
$posts = $user->posts;  // Collection (cached after first access)
$posts = $user->posts;  // Returns same cached Collection

// Method access - returns query builder
$query = $user->posts();  // Returns HasMany relation (builder)
$query = $user->posts()->where('published', true)->get();
```

## Constraining Eager Loads

```php
// Load only published posts
$users = User::with(['posts' => function ($query) {
    $query->where('published', true)
          ->orderBy('created_at', 'desc');
}])->get();

// Shorthand for simple constraints
$users = User::with('posts:id,user_id,title')->get();  // Select specific columns
```

## Counting Related Records

```php
// Eager load count
$users = User::withCount('posts')->get();

foreach ($users as $user) {
    echo $user->posts_count;  // Number of posts
}

// Count with constraints
$users = User::withCount(['posts' => function ($query) {
    $query->where('published', true);
}])->get();

foreach ($users as $user) {
    echo $user->posts_count;  // Published posts only
}

// Multiple counts
$users = User::withCount(['posts', 'comments'])->get();
// $user->posts_count, $user->comments_count
```

## Checking Existence

```php
// Has at least one post
$usersWithPosts = User::has('posts')->get();

// Has at least 5 posts
$activeAuthors = User::has('posts', '>=', 5)->get();

// Has posts matching condition
$usersWithPublished = User::whereHas('posts', function ($query) {
    $query->where('published', true);
})->get();

// Doesn't have posts
$usersWithoutPosts = User::doesntHave('posts')->get();
```

## Resources

- [One-to-Many Relationships](https://laravel.com/docs/12.x/eloquent-relationships#one-to-many) — Official documentation on one-to-many relationships

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*