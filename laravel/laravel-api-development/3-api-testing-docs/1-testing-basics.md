---
source_course: "laravel-api-development"
source_lesson: "laravel-api-development-api-testing-basics"
---

# Testing API Endpoints

Testing your API ensures it works correctly and continues to work as you make changes. Laravel provides excellent tools for API testing.

## Setting Up API Tests

Create a test file:

```bash
php artisan make:test Api/PostTest
```

This creates `tests/Feature/Api/PostTest.php`:

```php
<?php

namespace Tests\Feature\Api;

use App\Models\Post;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PostTest extends TestCase
{
    use RefreshDatabase;

    protected User $user;

    protected function setUp(): void
    {
        parent::setUp();
        $this->user = User::factory()->create();
    }
}
```

## Testing JSON Responses

### GET Requests

```php
public function test_can_list_posts(): void
{
    $posts = Post::factory()->count(3)->create();

    $response = $this->getJson('/api/posts');

    $response->assertStatus(200)
        ->assertJsonCount(3, 'data')
        ->assertJsonStructure([
            'data' => [
                '*' => ['id', 'title', 'body', 'created_at'],
            ],
        ]);
}

public function test_can_show_single_post(): void
{
    $post = Post::factory()->create();

    $response = $this->getJson("/api/posts/{$post->id}");

    $response->assertStatus(200)
        ->assertJson([
            'data' => [
                'id' => $post->id,
                'title' => $post->title,
            ],
        ]);
}
```

### POST Requests

```php
public function test_can_create_post(): void
{
    $data = [
        'title' => 'Test Post',
        'body' => 'This is the body content.',
    ];

    $response = $this->actingAs($this->user, 'sanctum')
        ->postJson('/api/posts', $data);

    $response->assertStatus(201)
        ->assertJson([
            'data' => [
                'title' => 'Test Post',
            ],
        ]);

    $this->assertDatabaseHas('posts', [
        'title' => 'Test Post',
        'user_id' => $this->user->id,
    ]);
}
```

### PUT/PATCH Requests

```php
public function test_can_update_post(): void
{
    $post = Post::factory()->for($this->user)->create();

    $response = $this->actingAs($this->user, 'sanctum')
        ->putJson("/api/posts/{$post->id}", [
            'title' => 'Updated Title',
        ]);

    $response->assertStatus(200)
        ->assertJson([
            'data' => [
                'title' => 'Updated Title',
            ],
        ]);

    $this->assertDatabaseHas('posts', [
        'id' => $post->id,
        'title' => 'Updated Title',
    ]);
}
```

### DELETE Requests

```php
public function test_can_delete_post(): void
{
    $post = Post::factory()->for($this->user)->create();

    $response = $this->actingAs($this->user, 'sanctum')
        ->deleteJson("/api/posts/{$post->id}");

    $response->assertStatus(204);

    $this->assertDatabaseMissing('posts', ['id' => $post->id]);
}
```

## Testing Authentication

```php
public function test_unauthenticated_user_cannot_create_post(): void
{
    $response = $this->postJson('/api/posts', [
        'title' => 'Test',
        'body' => 'Body',
    ]);

    $response->assertStatus(401);
}

public function test_user_cannot_update_others_post(): void
{
    $otherUser = User::factory()->create();
    $post = Post::factory()->for($otherUser)->create();

    $response = $this->actingAs($this->user, 'sanctum')
        ->putJson("/api/posts/{$post->id}", [
            'title' => 'Hacked!',
        ]);

    $response->assertStatus(403);
}
```

## Testing Validation

```php
public function test_post_creation_requires_title(): void
{
    $response = $this->actingAs($this->user, 'sanctum')
        ->postJson('/api/posts', [
            'body' => 'Body without title',
        ]);

    $response->assertStatus(422)
        ->assertJsonValidationErrors(['title']);
}

public function test_post_title_must_be_string(): void
{
    $response = $this->actingAs($this->user, 'sanctum')
        ->postJson('/api/posts', [
            'title' => 12345,
            'body' => 'Content',
        ]);

    $response->assertStatus(422)
        ->assertJsonValidationErrors(['title']);
}
```

## Testing with Sanctum Tokens

```php
public function test_can_authenticate_with_token(): void
{
    $token = $this->user->createToken('test-token')->plainTextToken;

    $response = $this->withHeader('Authorization', "Bearer {$token}")
        ->getJson('/api/user');

    $response->assertStatus(200)
        ->assertJson([
            'id' => $this->user->id,
            'email' => $this->user->email,
        ]);
}

public function test_token_with_limited_abilities(): void
{
    $token = $this->user->createToken('read-only', ['posts:read'])->plainTextToken;

    // Can read
    $response = $this->withHeader('Authorization', "Bearer {$token}")
        ->getJson('/api/posts');
    $response->assertStatus(200);

    // Cannot create
    $response = $this->withHeader('Authorization', "Bearer {$token}")
        ->postJson('/api/posts', ['title' => 'Test', 'body' => 'Body']);
    $response->assertStatus(403);
}
```

## JSON Assertion Methods

```php
// Assert exact JSON
$response->assertExactJson(['key' => 'value']);

// Assert JSON fragment
$response->assertJson(['key' => 'value']);

// Assert JSON structure
$response->assertJsonStructure([
    'data' => ['id', 'title'],
    'meta' => ['current_page', 'total'],
]);

// Assert JSON path
$response->assertJsonPath('data.0.title', 'Expected Title');

// Assert JSON count
$response->assertJsonCount(5, 'data');

// Assert validation errors
$response->assertJsonValidationErrors(['field']);
$response->assertJsonMissingValidationErrors(['field']);
```

## Resources

- [HTTP Tests](https://laravel.com/docs/12.x/http-tests) — Official documentation on HTTP testing

---

> 📘 *This lesson is part of the [Laravel API Development](https://stanza.dev/courses/laravel-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*