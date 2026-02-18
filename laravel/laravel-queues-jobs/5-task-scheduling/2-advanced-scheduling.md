---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-advanced-scheduling"
---

# Advanced Scheduling Features

Laravel's scheduler includes powerful features for controlling when and how tasks run.

## Preventing Task Overlaps

```php
Schedule::command('emails:send')
    ->withoutOverlapping()  // Skip if previous still running
    ->daily();

// With lock expiration
Schedule::command('reports:generate')
    ->withoutOverlapping(10)  // Lock expires after 10 minutes
    ->hourly();
```

## Running on One Server

For multi-server deployments:

```php
Schedule::command('report:generate')
    ->onOneServer()  // Only one server runs this
    ->daily();
```

Requires a cache driver that supports locks (Redis, Memcached, database).

## Background Tasks

```php
// Run in background (don't block other tasks)
Schedule::command('analytics:import')
    ->runInBackground()
    ->daily();
```

## Conditional Scheduling

```php
// Only run when condition is true
Schedule::command('emails:send')
    ->daily()
    ->when(function () {
        return config('mail.enabled');
    });

// Skip when condition is true
Schedule::command('backup:run')
    ->daily()
    ->skip(function () {
        return app()->isDownForMaintenance();
    });

// Only in certain environments
Schedule::command('telescope:prune')
    ->environments(['staging', 'production'])
    ->daily();
```

## Day Constraints

```php
// Only on weekdays
Schedule::command('report:generate')
    ->weekdays()
    ->at('08:00');

// Only on weekends
Schedule::command('cleanup:run')
    ->weekends()
    ->at('02:00');

// Specific days
Schedule::command('task')
    ->days([Schedule::MONDAY, Schedule::WEDNESDAY, Schedule::FRIDAY])
    ->at('09:00');
```

## Time Constraints

```php
// Only between certain hours
Schedule::command('process:queue')
    ->everyMinute()
    ->between('8:00', '17:00');  // Business hours

// Except between certain hours
Schedule::command('heavy:task')
    ->hourly()
    ->unlessBetween('9:00', '17:00');  // Not during business hours
```

## Task Output

```php
// Write output to file
Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo('/var/log/email-output.log');

// Append to file
Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo('/var/log/email-output.log');

// Email output
Schedule::command('report:generate')
    ->daily()
    ->emailOutputTo('admin@example.com');

// Email only on failure
Schedule::command('backup:run')
    ->daily()
    ->emailOutputOnFailure('admin@example.com');
```

## Task Hooks

```php
Schedule::command('emails:send')
    ->daily()
    ->before(function () {
        Log::info('Starting email send...');
    })
    ->after(function () {
        Log::info('Email send complete.');
    })
    ->onSuccess(function () {
        Notification::send($admin, new TaskSucceeded('emails:send'));
    })
    ->onFailure(function () {
        Notification::send($admin, new TaskFailed('emails:send'));
    });
```

## Ping URLs (Heartbeats)

```php
// Ping URL before/after task
Schedule::command('backup:run')
    ->daily()
    ->pingBefore('https://healthchecks.io/ping/start')
    ->thenPing('https://healthchecks.io/ping/end')
    ->pingOnSuccess('https://healthchecks.io/ping/success')
    ->pingOnFailure('https://healthchecks.io/ping/failure');
```

## Viewing Scheduled Tasks

```bash
# List all scheduled tasks
php artisan schedule:list

# Output:
# +--------------------+-----------+-------------+------------------+
# | Command            | Interval  | Next Due    | Description      |
# +--------------------+-----------+-------------+------------------+
# | emails:send        | Daily     | 2024-01-16  | Send emails      |
# | backup:run         | Daily 2am | 2024-01-16  | Database backup  |
# +--------------------+-----------+-------------+------------------+
```

## Complete Example

```php
// routes/console.php
use Illuminate\Support\Facades\Schedule;

// Daily tasks
Schedule::command('backup:clean')->daily()->at('01:00');
Schedule::command('backup:run')->daily()->at('02:00');

// Email reports
Schedule::command('report:daily')
    ->dailyAt('08:00')
    ->weekdays()
    ->emailOutputOnFailure('admin@example.com');

// Cleanup
Schedule::command('telescope:prune --hours=48')
    ->daily()
    ->environments(['production']);

Schedule::command('queue:prune-failed --hours=168')
    ->weekly();

// Processing
Schedule::command('process:pending-orders')
    ->everyFiveMinutes()
    ->withoutOverlapping()
    ->onOneServer();

// Maintenance (off-hours only)
Schedule::command('db:optimize')
    ->weekly()
    ->sundays()
    ->at('03:00')
    ->unlessBetween('08:00', '22:00');
```

## Resources

- [Scheduling](https://laravel.com/docs/12.x/scheduling) — Complete scheduling documentation

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*