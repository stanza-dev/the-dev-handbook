---
source_course: "laravel-testing"
source_lesson: "laravel-testing-testing-best-practices"
---

# Testing Best Practices

Write maintainable, reliable tests that give you confidence in your code.

## AAA Pattern: Arrange, Act, Assert

```php
public function test_user_can_publish_post(): void
{
    // Arrange - Set up the test
    $user = User::factory()->create();
    $post = Post::factory()->draft()->create(['user_id' => $user->id]);

    // Act - Perform the action
    $response = $this->actingAs($user)
        ->post("/posts/{$post->id}/publish");

    // Assert - Verify the results
    $response->assertRedirect();
    $this->assertNotNull($post->fresh()->published_at);
}
```

## One Assertion per Test (When Practical)

```php
// ❌ Too many unrelated assertions
public function test_post_creation(): void
{
    $response = $this->actingAs($this->user)
        ->post('/posts', $this->validData());

    $response->assertRedirect();
    $this->assertDatabaseCount('posts', 1);
    $this->assertTrue($this->user->posts()->exists());
    Mail::assertSent(PostCreatedNotification::class);
    Event::assertDispatched(PostCreated::class);
}

// ✅ Focused tests
public function test_successful_post_creation_redirects(): void
{
    $response = $this->actingAs($this->user)
        ->post('/posts', $this->validData());

    $response->assertRedirect('/posts');
}

public function test_post_is_stored_in_database(): void
{
    $this->actingAs($this->user)
        ->post('/posts', $this->validData());

    $this->assertDatabaseHas('posts', [
        'title' => 'Test Post',
        'user_id' => $this->user->id,
    ]);
}

public function test_notification_is_sent_on_post_creation(): void
{
    Mail::fake();

    $this->actingAs($this->user)
        ->post('/posts', $this->validData());

    Mail::assertSent(PostCreatedNotification::class);
}
```

## Descriptive Test Names

```php
// ❌ Vague names
public function test_post(): void
public function test_validation(): void
public function test_it_works(): void

// ✅ Descriptive names
public function test_authenticated_users_can_create_posts(): void
public function test_post_title_is_required(): void
public function test_guests_are_redirected_to_login_when_creating_posts(): void

// With Pest - even more readable
test('authenticated users can create posts', function () { });
test('post title is required', function () { });
test('guests are redirected to login', function () { });
```

## Use Data Providers for Variations

```php
/**
 * @dataProvider invalidPostData
 */
public function test_post_validation(string $field, mixed $value, string $error): void
{
    $data = $this->validData();
    $data[$field] = $value;

    $response = $this->actingAs($this->user)
        ->post('/posts', $data);

    $response->assertSessionHasErrors([$field => $error]);
}

public static function invalidPostData(): array
{
    return [
        'title is required' => ['title', '', 'The title field is required.'],
        'title max length' => ['title', str_repeat('a', 256), 'The title must not exceed 255 characters.'],
        'body is required' => ['body', '', 'The body field is required.'],
        'category must exist' => ['category_id', 999, 'The selected category is invalid.'],
    ];
}
```

With Pest:

```php
it('validates post data', function (string $field, mixed $value, string $error) {
    $data = validData();
    $data[$field] = $value;

    $this->actingAs($this->user)
        ->post('/posts', $data)
        ->assertSessionHasErrors([$field => $error]);
})->with([
    'title required' => ['title', '', 'The title field is required.'],
    'title max' => ['title', str_repeat('a', 256), 'max 255'],
]);
```

## Setup Common Test Data

```php
class PostTest extends TestCase
{
    use RefreshDatabase;

    protected User $user;
    protected Category $category;

    protected function setUp(): void
    {
        parent::setUp();

        $this->user = User::factory()->create();
        $this->category = Category::factory()->create();
    }

    protected function validData(array $overrides = []): array
    {
        return array_merge([
            'title' => 'Test Post',
            'body' => 'This is the post body content.',
            'category_id' => $this->category->id,
        ], $overrides);
    }

    public function test_example(): void
    {
        $response = $this->actingAs($this->user)
            ->post('/posts', $this->validData(['title' => 'Custom Title']));

        $response->assertRedirect();
    }
}
```

## Test Edge Cases

```php
public function test_handles_empty_post_list(): void
{
    $response = $this->get('/posts');

    $response->assertOk();
    $response->assertSee('No posts found');
}

public function test_handles_very_long_content(): void
{
    $post = Post::factory()->create([
        'body' => str_repeat('a', 100000),
    ]);

    $response = $this->get("/posts/{$post->id}");

    $response->assertOk();
}

public function test_handles_special_characters_in_title(): void
{
    $post = Post::factory()->create([
        'title' => 'Test <script>alert("XSS")</script> & "quotes"',
    ]);

    $response = $this->get('/posts');

    $response->assertSee(e($post->title), false);
    $response->assertDontSee('<script>');
}
```

## Don't Test Framework Code

```php
// ❌ Don't test Laravel's validation rules
public function test_email_validation(): void
{
    // Laravel already tests this
}

// ✅ Test YOUR validation rules are applied
public function test_email_is_required(): void
{
    $response = $this->post('/register', ['email' => '']);
    $response->assertSessionHasErrors('email');
}
```

## Keep Tests Fast

```php
// ❌ Slow: Creates 100 users
public function test_user_list_shows_pagination(): void
{
    User::factory()->count(100)->create();
    // ...
}

// ✅ Fast: Only create what you need
public function test_user_list_shows_pagination(): void
{
    User::factory()->count(16)->create();  // Just past 15/page
    // ...
}
```

## Resources

- [Testing Best Practices](https://laravel.com/docs/12.x/testing) — Laravel testing documentation

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*