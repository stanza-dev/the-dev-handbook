---
source_course: "laravel-testing"
source_lesson: "laravel-testing-http-testing"
---

# HTTP and Feature Testing

Feature tests allow you to test your application by making HTTP requests and asserting on the responses. They test how multiple components work together.

## Making HTTP Requests

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class PostTest extends TestCase
{
    public function test_users_can_view_posts(): void
    {
        $response = $this->get('/posts');

        $response->assertStatus(200);
    }

    public function test_users_can_create_posts(): void
    {
        $response = $this->post('/posts', [
            'title' => 'Test Post',
            'body' => 'This is a test post body.',
        ]);

        $response->assertStatus(201);
    }
}
```

## All HTTP Methods

```php
$this->get('/posts');
$this->post('/posts', $data);
$this->put('/posts/1', $data);
$this->patch('/posts/1', $data);
$this->delete('/posts/1');

// With headers
$this->withHeaders([
    'X-Custom-Header' => 'Value',
])->get('/api/posts');

// JSON requests (sets Accept: application/json)
$this->getJson('/api/posts');
$this->postJson('/api/posts', $data);
$this->putJson('/api/posts/1', $data);
$this->deleteJson('/api/posts/1');
```

## Response Assertions

### Status Codes

```php
$response->assertStatus(200);
$response->assertOk();           // 200
$response->assertCreated();      // 201
$response->assertNoContent();    // 204
$response->assertNotFound();     // 404
$response->assertForbidden();    // 403
$response->assertUnauthorized(); // 401
$response->assertUnprocessable(); // 422
```

### Response Content

```php
// View assertions
$response->assertViewIs('posts.index');
$response->assertViewHas('posts');
$response->assertViewHas('posts', $expectedPosts);
$response->assertViewHas('user', function ($user) {
    return $user->name === 'John';
});

// Text assertions
$response->assertSee('Welcome');
$response->assertDontSee('Error');
$response->assertSeeText('Welcome');  // Ignores HTML tags

// JSON assertions
$response->assertJson([
    'id' => 1,
    'title' => 'Test Post',
]);

$response->assertJsonPath('data.0.title', 'First Post');
$response->assertJsonCount(3, 'data');
$response->assertJsonStructure([
    'data' => [
        '*' => ['id', 'title', 'body'],
    ],
    'meta' => ['total', 'per_page'],
]);

$response->assertExactJson([...]);  // Must match exactly
$response->assertJsonMissing(['secret_field']);
```

### Redirects

```php
$response->assertRedirect('/dashboard');
$response->assertRedirectToRoute('dashboard');
$response->assertRedirectToSignedRoute('unsubscribe');
```

### Headers and Cookies

```php
$response->assertHeader('Content-Type', 'application/json');
$response->assertHeaderMissing('X-Secret-Header');
$response->assertCookie('session_id');
$response->assertCookieExpired('old_cookie');
```

## Testing with Authentication

```php
use App\Models\User;

public function test_authenticated_users_can_create_posts(): void
{
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->post('/posts', [
            'title' => 'Test Post',
            'body' => 'Content here...',
        ]);

    $response->assertCreated();
}

public function test_guests_cannot_create_posts(): void
{
    $response = $this->post('/posts', [
        'title' => 'Test Post',
        'body' => 'Content here...',
    ]);

    $response->assertRedirect('/login');
}

// API authentication with Sanctum
public function test_api_authentication(): void
{
    $user = User::factory()->create();

    $response = $this->actingAs($user, 'sanctum')
        ->getJson('/api/user');

    $response->assertOk();
    $response->assertJson(['id' => $user->id]);
}
```

## Session and Flash Data

```php
// Set session before request
$this->withSession(['key' => 'value'])
    ->get('/dashboard');

// Assert session
$response->assertSessionHas('status', 'success');
$response->assertSessionHasErrors(['email', 'password']);
$response->assertSessionHasErrors(['email' => 'Invalid email']);
$response->assertSessionDoesntHaveErrors();
```

## Testing File Uploads

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

public function test_users_can_upload_avatar(): void
{
    Storage::fake('avatars');

    $file = UploadedFile::fake()->image('avatar.jpg');

    $response = $this->actingAs($this->user)
        ->post('/avatar', [
            'avatar' => $file,
        ]);

    $response->assertOk();

    // Assert file was stored
    Storage::disk('avatars')->assertExists($file->hashName());
}

public function test_only_images_are_accepted(): void
{
    Storage::fake('avatars');

    $file = UploadedFile::fake()->create('document.pdf', 100);

    $response = $this->actingAs($this->user)
        ->post('/avatar', ['avatar' => $file]);

    $response->assertSessionHasErrors(['avatar']);
}
```

## Complete Feature Test Example

```php
<?php

namespace Tests\Feature;

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

    public function test_guests_can_view_posts(): void
    {
        $post = Post::factory()->create(['title' => 'Test Post']);

        $response = $this->get('/posts');

        $response->assertOk();
        $response->assertSee('Test Post');
    }

    public function test_authenticated_users_can_create_posts(): void
    {
        $response = $this->actingAs($this->user)
            ->post('/posts', [
                'title' => 'New Post',
                'body' => 'Post content here...',
            ]);

        $response->assertRedirect('/posts');
        $this->assertDatabaseHas('posts', [
            'title' => 'New Post',
            'user_id' => $this->user->id,
        ]);
    }

    public function test_post_title_is_required(): void
    {
        $response = $this->actingAs($this->user)
            ->post('/posts', [
                'body' => 'Post content here...',
            ]);

        $response->assertSessionHasErrors(['title']);
    }

    public function test_users_can_only_update_their_own_posts(): void
    {
        $post = Post::factory()->create();  // Different user

        $response = $this->actingAs($this->user)
            ->put("/posts/{$post->id}", [
                'title' => 'Updated Title',
            ]);

        $response->assertForbidden();
    }
}
```

## Resources

- [HTTP Tests](https://laravel.com/docs/12.x/http-tests) — Official HTTP testing documentation

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*