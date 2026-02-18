---
source_course: "laravel-api-development"
source_lesson: "laravel-api-development-rate-limiting"
---

# API Rate Limiting

Rate limiting protects your API from abuse by limiting how many requests a client can make in a given time period.

## Default Rate Limiting

Laravel's API routes are rate-limited by default. Configure in `AppServiceProvider`:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

## Custom Rate Limiters

```php
public function boot(): void
{
    // Per minute limit
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });

    // Different limits for authenticated vs guest
    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()
            ? Limit::perMinute(100)->by($request->user()->id)
            : Limit::perMinute(10)->by($request->ip());
    });

    // Premium users get unlimited
    RateLimiter::for('premium', function (Request $request) {
        return $request->user()?->isPremium()
            ? Limit::none()
            : Limit::perMinute(30)->by($request->user()?->id ?: $request->ip());
    });

    // Per hour limit
    RateLimiter::for('reports', function (Request $request) {
        return Limit::perHour(10)->by($request->user()->id);
    });

    // Per day limit
    RateLimiter::for('exports', function (Request $request) {
        return Limit::perDay(5)->by($request->user()->id);
    });
}
```

## Applying Rate Limiters

```php
// Using the throttle middleware
Route::middleware('throttle:api')->group(function () {
    Route::get('/posts', [PostController::class, 'index']);
});

// Custom rate limiter
Route::middleware('throttle:uploads')->group(function () {
    Route::post('/upload', [UploadController::class, 'store']);
});

// Multiple rate limiters
Route::middleware(['throttle:api', 'throttle:uploads'])->group(function () {
    // Both limits apply
});
```

## Rate Limit Headers

Laravel automatically includes rate limit headers:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
Retry-After: 60 (when limit exceeded)
```

## Custom Response on Limit Exceeded

```php
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)
        ->by($request->user()?->id ?: $request->ip())
        ->response(function (Request $request, array $headers) {
            return response()->json([
                'success' => false,
                'message' => 'Too many requests. Please slow down.',
                'retry_after' => $headers['Retry-After'],
            ], 429, $headers);
        });
});
```

## Segmented Rate Limiting

Different limits for different endpoints:

```php
// Heavy operations
RateLimiter::for('heavy', function (Request $request) {
    return Limit::perMinute(10)->by($request->user()->id);
});

// Light operations
RateLimiter::for('light', function (Request $request) {
    return Limit::perMinute(120)->by($request->user()->id);
});
```

```php
Route::middleware('throttle:light')->group(function () {
    Route::get('/posts', [PostController::class, 'index']);
    Route::get('/posts/{post}', [PostController::class, 'show']);
});

Route::middleware('throttle:heavy')->group(function () {
    Route::post('/analyze', [AnalyzeController::class, 'store']);
    Route::post('/export', [ExportController::class, 'store']);
});
```

## Multiple Limits

```php
RateLimiter::for('api', function (Request $request) {
    return [
        // 60 per minute
        Limit::perMinute(60)->by($request->user()?->id ?: $request->ip()),
        // 1000 per day
        Limit::perDay(1000)->by($request->user()?->id ?: $request->ip()),
    ];
});
```

## Manual Rate Limiting

```php
use Illuminate\Support\Facades\RateLimiter;

public function sendVerificationCode(Request $request)
{
    $key = 'verify-phone:' . $request->user()->id;

    if (RateLimiter::tooManyAttempts($key, 3)) {
        $seconds = RateLimiter::availableIn($key);

        return response()->json([
            'message' => "Too many attempts. Try again in {$seconds} seconds.",
        ], 429);
    }

    RateLimiter::hit($key, 60);  // Decay in 60 seconds

    // Send verification code...

    return response()->json(['message' => 'Code sent!']);
}

// Clear attempts (e.g., after successful verification)
RateLimiter::clear('verify-phone:' . $user->id);
```

## Testing Rate Limits

```php
public function test_rate_limiting_works()
{
    // Make 61 requests (limit is 60)
    for ($i = 0; $i < 61; $i++) {
        $response = $this->getJson('/api/posts');
    }

    // 61st request should be rate limited
    $response->assertStatus(429);
    $response->assertJson([
        'message' => 'Too many requests.',
    ]);
}
```

## Resources

- [Rate Limiting](https://laravel.com/docs/12.x/routing#rate-limiting) — Official rate limiting documentation

---

> 📘 *This lesson is part of the [Laravel API Development](https://stanza.dev/courses/laravel-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*