---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-crud-operations"
---

# CRUD Operations with Eloquent

Learn how to Create, Read, Update, and Delete records using Eloquent's expressive API.

## Create Operations

### Using save()

```php
// Create instance, set properties, save
$post = new Post;
$post->title = 'My First Post';
$post->body = 'Post content here...';
$post->user_id = auth()->id();
$post->save();

// Now it has an ID
echo $post->id;  // 1
```

### Using create()

```php
// Create and save in one step
$post = Post::create([
    'title' => 'My First Post',
    'body' => 'Post content here...',
    'user_id' => auth()->id(),
]);
```

**Note**: `create()` requires the attributes to be in `$fillable`.

### firstOrCreate and firstOrNew

```php
// Find or create (saves to database)
$user = User::firstOrCreate(
    ['email' => 'john@example.com'],      // Search criteria
    ['name' => 'John', 'password' => ...] // Additional fields if creating
);

// Find or instantiate (doesn't save)
$user = User::firstOrNew(
    ['email' => 'john@example.com'],
    ['name' => 'John']
);
$user->save();  // Save manually
```

### updateOrCreate (Upsert)

```php
// Update if exists, create if not
$post = Post::updateOrCreate(
    ['slug' => 'my-post'],           // Search criteria
    ['title' => 'Updated Title', ...] // Values to update/create
);
```

## Read Operations

### Retrieving Single Records

```php
// Find by primary key
$post = Post::find(1);

// Find multiple by primary key
$posts = Post::find([1, 2, 3]);

// Find or fail (throws ModelNotFoundException)
$post = Post::findOrFail(1);

// Find or 404
Route::get('/posts/{id}', function ($id) {
    $post = Post::findOrFail($id);  // Auto 404 if not found
    return view('posts.show', compact('post'));
});

// First matching record
$post = Post::where('slug', 'my-post')->first();
$post = Post::where('slug', 'my-post')->firstOrFail();
```

### Retrieving Multiple Records

```php
// All records
$posts = Post::all();

// With constraints
$posts = Post::where('published', true)->get();

// Multiple conditions
$posts = Post::where('published', true)
    ->where('category_id', 1)
    ->orderBy('created_at', 'desc')
    ->get();

// Or conditions
$posts = Post::where('featured', true)
    ->orWhere('views', '>', 1000)
    ->get();
```

### Chunking for Large Datasets

```php
// Process 100 records at a time
Post::chunk(100, function ($posts) {
    foreach ($posts as $post) {
        // Process each post
    }
});

// Using lazy() for memory efficiency
foreach (Post::lazy() as $post) {
    // Process one at a time
}

// Cursor for ultimate memory efficiency
foreach (Post::cursor() as $post) {
    // Hydrates one model at a time
}
```

### Aggregates

```php
$count = Post::count();
$max = Post::max('views');
$min = Post::min('views');
$avg = Post::avg('views');
$sum = Post::sum('views');

$exists = Post::where('slug', 'my-post')->exists();
$doesntExist = Post::where('slug', 'my-post')->doesntExist();
```

## Update Operations

### Updating Single Records

```php
$post = Post::find(1);
$post->title = 'Updated Title';
$post->save();

// Or in one line
$post = Post::find(1);
$post->update(['title' => 'Updated Title']);
```

### Mass Updates

```php
// Update all matching records
Post::where('published', false)
    ->update(['status' => 'draft']);

// Increment/decrement
Post::where('id', 1)->increment('views');
Post::where('id', 1)->increment('views', 5);  // By 5
Post::where('id', 1)->decrement('stock');

// With additional updates
$post->increment('views', 1, ['last_viewed_at' => now()]);
```

### Checking for Changes

```php
$post = Post::find(1);
$post->title = 'New Title';

$post->isDirty();           // true - has unsaved changes
$post->isDirty('title');    // true
$post->isDirty('body');     // false
$post->isClean();           // false

$post->save();

$post->wasChanged();        // true - was just saved with changes
$post->wasChanged('title'); // true
```

## Delete Operations

```php
// Delete a single model
$post = Post::find(1);
$post->delete();

// Delete by primary key
Post::destroy(1);
Post::destroy([1, 2, 3]);
Post::destroy(1, 2, 3);

// Delete matching records
Post::where('published', false)->delete();

// Truncate (delete all)
Post::truncate();  // Warning: No events fired!
```

## Soft Deletes

Keep records but mark as deleted:

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes;
}
```

Requires `deleted_at` column in migration:

```php
$table->softDeletes();  // Adds deleted_at column
```

Usage:

```php
$post->delete();  // Sets deleted_at, doesn't remove row

// Query soft deleted records
Post::withTrashed()->get();      // Include deleted
Post::onlyTrashed()->get();      // Only deleted

// Restore
$post->restore();

// Permanently delete
$post->forceDelete();
```

## Resources

- [Inserting & Updating Models](https://laravel.com/docs/12.x/eloquent#inserting-and-updating-models) — Official documentation on CRUD operations

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*