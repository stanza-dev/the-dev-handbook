---
source_course: "laravel-testing"
source_lesson: "laravel-testing-mocking-basics"
---

# Mocking and Faking in Laravel

Mocking allows you to replace dependencies with test doubles, isolating the code under test. Laravel provides convenient fakes for many services.

## Why Mock?

```
Without Mocking:              With Mocking:
┌──────────────┐              ┌──────────────┐
│   Your Test  │              │   Your Test  │
└──────────────┘              └──────────────┘
       │                             │
       ▼                             ▼
┌──────────────┐              ┌──────────────┐
│ Your Service │              │ Your Service │
└──────────────┘              └──────────────┘
       │                             │
       ▼                             ▼
┌──────────────┐              ┌──────────────┐
│ Real Email   │              │  Fake Email  │
│   Server     │              │   (No send)  │
└──────────────┘              └──────────────┘
   Slow, Side effects            Fast, Isolated
```

## Built-in Fakes

Laravel provides fakes for common services:

### Mail Fake

```php
use Illuminate\Support\Facades\Mail;
use App\Mail\WelcomeEmail;

public function test_welcome_email_is_sent(): void
{
    Mail::fake();

    // Perform action that sends email
    $user = User::factory()->create();

    // Assert email was sent
    Mail::assertSent(WelcomeEmail::class);

    // Assert sent to specific user
    Mail::assertSent(WelcomeEmail::class, function ($mail) use ($user) {
        return $mail->hasTo($user->email);
    });

    // Assert sent count
    Mail::assertSent(WelcomeEmail::class, 1);

    // Assert not sent
    Mail::assertNotSent(PasswordResetEmail::class);

    // Assert nothing was sent
    Mail::assertNothingSent();
}
```

### Notification Fake

```php
use Illuminate\Support\Facades\Notification;
use App\Notifications\InvoicePaid;

public function test_notification_is_sent(): void
{
    Notification::fake();

    $user = User::factory()->create();
    $invoice = Invoice::factory()->create(['user_id' => $user->id]);

    // Perform action
    $invoice->markAsPaid();

    // Assert notification sent to user
    Notification::assertSentTo($user, InvoicePaid::class);

    // Assert with conditions
    Notification::assertSentTo(
        $user,
        InvoicePaid::class,
        function ($notification, $channels) use ($invoice) {
            return $notification->invoice->id === $invoice->id;
        }
    );

    // Assert not sent
    Notification::assertNotSentTo($user, OtherNotification::class);
}
```

### Event Fake

```php
use Illuminate\Support\Facades\Event;
use App\Events\OrderShipped;

public function test_order_shipped_event_is_dispatched(): void
{
    Event::fake();

    // Perform action
    $order = Order::factory()->create();
    $order->ship();

    // Assert event dispatched
    Event::assertDispatched(OrderShipped::class);

    // Assert with conditions
    Event::assertDispatched(OrderShipped::class, function ($event) use ($order) {
        return $event->order->id === $order->id;
    });

    // Assert count
    Event::assertDispatched(OrderShipped::class, 1);

    // Fake specific events only
    Event::fake([OrderShipped::class]);
}
```

### Queue Fake

```php
use Illuminate\Support\Facades\Queue;
use App\Jobs\ProcessPodcast;

public function test_job_is_queued(): void
{
    Queue::fake();

    // Perform action
    ProcessPodcast::dispatch($podcast);

    // Assert job was pushed
    Queue::assertPushed(ProcessPodcast::class);

    // Assert pushed to specific queue
    Queue::assertPushedOn('podcasts', ProcessPodcast::class);

    // Assert with conditions
    Queue::assertPushed(ProcessPodcast::class, function ($job) use ($podcast) {
        return $job->podcast->id === $podcast->id;
    });

    // Assert chain
    Queue::assertPushedWithChain(ProcessPodcast::class, [
        OptimizePodcast::class,
        PublishPodcast::class,
    ]);
}
```

### Storage Fake

```php
use Illuminate\Support\Facades\Storage;
use Illuminate\Http\UploadedFile;

public function test_file_is_uploaded(): void
{
    Storage::fake('avatars');

    $file = UploadedFile::fake()->image('avatar.jpg');

    $this->actingAs($user)
        ->post('/avatar', ['avatar' => $file]);

    // Assert file exists
    Storage::disk('avatars')->assertExists('avatars/' . $file->hashName());

    // Assert file doesn't exist
    Storage::disk('avatars')->assertMissing('missing.jpg');

    // Assert directory exists
    Storage::disk('avatars')->assertDirectoryEmpty('empty-folder');
}
```

### HTTP Fake

```php
use Illuminate\Support\Facades\Http;

public function test_external_api_call(): void
{
    Http::fake([
        'api.github.com/*' => Http::response(['login' => 'octocat'], 200),
        'api.stripe.com/*' => Http::response(['id' => 'ch_123'], 200),
    ]);

    // Your code makes HTTP requests
    $response = Http::get('https://api.github.com/users/1');

    // Assert requests were made
    Http::assertSent(function ($request) {
        return $request->url() === 'https://api.github.com/users/1';
    });

    // Assert request count
    Http::assertSentCount(1);

    // Sequence of responses
    Http::fake([
        '*' => Http::sequence()
            ->push(['status' => 'ok'], 200)
            ->push(['status' => 'error'], 500),
    ]);
}
```

## Mockery for Custom Mocking

```php
use Mockery;
use App\Services\PaymentGateway;

public function test_payment_is_processed(): void
{
    $mock = Mockery::mock(PaymentGateway::class);

    $mock->shouldReceive('charge')
        ->once()
        ->with(100, Mockery::type('string'))
        ->andReturn(['success' => true, 'id' => 'txn_123']);

    $this->app->instance(PaymentGateway::class, $mock);

    // Now when PaymentGateway is injected, the mock is used
    $response = $this->post('/checkout', ['amount' => 100]);

    $response->assertOk();
}

protected function tearDown(): void
{
    Mockery::close();
    parent::tearDown();
}
```

## Laravel's Mock Method

```php
use App\Services\PaymentGateway;

public function test_payment_with_laravel_mock(): void
{
    $this->mock(PaymentGateway::class, function ($mock) {
        $mock->shouldReceive('charge')
            ->once()
            ->andReturn(['success' => true]);
    });

    $response = $this->post('/checkout', ['amount' => 100]);

    $response->assertOk();
}

// Partial mock
$this->partialMock(PaymentGateway::class, function ($mock) {
    $mock->shouldReceive('charge')->andReturn(['success' => true]);
    // Other methods work normally
});

// Spy (verify after)
$spy = $this->spy(PaymentGateway::class);

$this->post('/checkout', ['amount' => 100]);

$spy->shouldHaveReceived('charge')->once();
```

## Resources

- [Mocking](https://laravel.com/docs/12.x/mocking) — Official mocking documentation

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*