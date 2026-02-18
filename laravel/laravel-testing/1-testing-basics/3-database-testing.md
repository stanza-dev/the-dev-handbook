---
source_course: "laravel-testing"
source_lesson: "laravel-testing-database-testing"
---

# Database Testing

Laravel provides powerful tools for testing database interactions, including migrations, factories, and seeders.

## RefreshDatabase Trait

Reset the database between tests:

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PostTest extends TestCase
{
    use RefreshDatabase;  // Resets database for each test

    public function test_posts_can_be_created(): void
    {
        // Database is fresh for this test
    }
}
```

**Options**:
- `RefreshDatabase` - Runs migrations once, uses transactions (fastest)
- `DatabaseMigrations` - Runs migrations before each test
- `DatabaseTransactions` - Wraps each test in a transaction

## Database Assertions

```php
use App\Models\Post;

public function test_post_is_stored_in_database(): void
{
    $user = User::factory()->create();

    $this->actingAs($user)->post('/posts', [
        'title' => 'Test Post',
        'body' => 'Content',
    ]);

    // Assert record exists
    $this->assertDatabaseHas('posts', [
        'title' => 'Test Post',
        'user_id' => $user->id,
    ]);

    // Assert record doesn't exist
    $this->assertDatabaseMissing('posts', [
        'title' => 'Nonexistent Post',
    ]);

    // Assert count
    $this->assertDatabaseCount('posts', 1);

    // Assert empty
    $this->assertDatabaseEmpty('comments');
}

public function test_soft_deleted_posts(): void
{
    $post = Post::factory()->create();
    $post->delete();

    // Record still exists (soft deleted)
    $this->assertSoftDeleted('posts', ['id' => $post->id]);

    // Or with model
    $this->assertSoftDeleted($post);

    // Not soft deleted
    $this->assertNotSoftDeleted('posts', ['id' => $post->id]);
}

public function test_model_was_deleted(): void
{
    $post = Post::factory()->create();
    $postId = $post->id;

    $this->actingAs($post->user)
        ->delete("/posts/{$postId}");

    $this->assertModelMissing($post);
}
```

## Model Factories

Factories generate test data:

```php
// Create a single model (persisted to database)
$user = User::factory()->create();

// Create multiple models
$users = User::factory()->count(3)->create();

// Create with specific attributes
$admin = User::factory()->create([
    'name' => 'Admin User',
    'is_admin' => true,
]);

// Make (without persisting)
$user = User::factory()->make();

// Create with relationships
$post = Post::factory()
    ->for(User::factory(), 'author')
    ->has(Comment::factory()->count(3))
    ->create();
```

## Creating Factories

```bash
php artisan make:factory PostFactory
```

```php
<?php

namespace Database\Factories;

use App\Models\Post;
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class PostFactory extends Factory
{
    protected $model = Post::class;

    public function definition(): array
    {
        return [
            'user_id' => User::factory(),
            'title' => fake()->sentence(),
            'slug' => fake()->slug(),
            'body' => fake()->paragraphs(3, true),
            'published_at' => fake()->optional()->dateTime(),
        ];
    }

    // States for different scenarios
    public function published(): static
    {
        return $this->state(fn (array $attributes) => [
            'published_at' => now(),
        ]);
    }

    public function draft(): static
    {
        return $this->state(fn (array $attributes) => [
            'published_at' => null,
        ]);
    }

    public function byUser(User $user): static
    {
        return $this->state(fn (array $attributes) => [
            'user_id' => $user->id,
        ]);
    }
}
```

Usage:

```php
// Using states
$post = Post::factory()->published()->create();
$draft = Post::factory()->draft()->create();

// Chaining states
$post = Post::factory()
    ->published()
    ->byUser($user)
    ->create();
```

## Factory Relationships

```php
// Create with related models
$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();

// Shorthand: hasPosts
$user = User::factory()
    ->hasPosts(3)
    ->create();

// With customized related models
$user = User::factory()
    ->has(
        Post::factory()
            ->count(3)
            ->state(['published_at' => now()])
    )
    ->create();

// Belonging to relationship
$posts = Post::factory()
    ->count(3)
    ->for(User::factory()->state(['name' => 'John']))
    ->create();

// Many-to-many
$post = Post::factory()
    ->hasAttached(
        Tag::factory()->count(3),
        ['added_by' => $user->id]  // Pivot data
    )
    ->create();
```

## Seeders in Tests

```php
use Database\Seeders\CategorySeeder;

public function test_posts_have_categories(): void
{
    // Run specific seeder
    $this->seed(CategorySeeder::class);

    // Or run DatabaseSeeder
    $this->seed();

    // Now test with seeded data
    $categories = Category::all();
    $this->assertGreaterThan(0, $categories->count());
}
```

## In-Memory SQLite

Fastest testing with in-memory database:

```xml
<!-- phpunit.xml -->
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
```

## Testing Model Events

```php
public function test_post_creates_slug_on_saving(): void
{
    $post = Post::factory()->create(['title' => 'My Test Post']);

    $this->assertEquals('my-test-post', $post->slug);
}

public function test_deleting_user_cascades_to_posts(): void
{
    $user = User::factory()->hasPosts(3)->create();
    $postIds = $user->posts->pluck('id');

    $user->delete();

    $postIds->each(fn ($id) => 
        $this->assertDatabaseMissing('posts', ['id' => $id])
    );
}
```

## Resources

- [Database Testing](https://laravel.com/docs/12.x/database-testing) — Official database testing documentation
- [Eloquent Factories](https://laravel.com/docs/12.x/eloquent-factories) — Model factory documentation

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*