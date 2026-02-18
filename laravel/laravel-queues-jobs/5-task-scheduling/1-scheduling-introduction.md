---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-scheduling-introduction"
---

# Introduction to Task Scheduling

Laravel's task scheduler allows you to define your command schedule within Laravel itself, requiring only a single cron entry on your server.

## Why Use the Scheduler?

```
Traditional Cron:                    Laravel Scheduler:
┌─────────────────────────────────┐  ┌─────────────────────────────────┐
│ # Server crontab (complex!)     │  │ # Server crontab (simple!)      │
│ 0 * * * * /path/to/report.php   │  │ * * * * * cd /app && artisan   │
│ */5 * * * * /path/to/cleanup.php│  │           schedule:run >> /log │
│ 0 0 * * * /path/to/backup.php   │  └─────────────────────────────────┘
│ 30 2 * * 0 /path/to/prune.php   │
│ ...many more entries...         │  All scheduling defined in PHP!
└─────────────────────────────────┘
```

## Defining Schedules

Define schedules in `routes/console.php`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->daily();
Schedule::command('reports:generate')->weeklyOn(1, '8:00');
Schedule::command('cache:clear')->hourly();
```

## Scheduling Artisan Commands

```php
// Run an Artisan command
Schedule::command('emails:send --force')->daily();

// With arguments
Schedule::command('user:cleanup', ['--days' => 30])->weekly();

// Signature style
Schedule::command('inspire')->hourly();
```

## Scheduling Queued Jobs

```php
use App\Jobs\ProcessReports;
use App\Jobs\CleanupDatabase;

Schedule::job(new ProcessReports)->daily();
Schedule::job(new CleanupDatabase, 'maintenance')->weekly();
```

## Scheduling Shell Commands

```php
Schedule::exec('node /home/user/scripts/backup.js')->daily();
Schedule::exec('mysqldump database > backup.sql')->dailyAt('02:00');
```

## Scheduling Closures

```php
Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();

Schedule::call(fn () => cache()->flush())->weekly();
```

## Schedule Frequency Options

```php
// Time-based
Schedule::command('task')->everyMinute();
Schedule::command('task')->everyTwoMinutes();
Schedule::command('task')->everyFiveMinutes();
Schedule::command('task')->everyTenMinutes();
Schedule::command('task')->everyFifteenMinutes();
Schedule::command('task')->everyThirtyMinutes();
Schedule::command('task')->hourly();
Schedule::command('task')->hourlyAt(17);  // At :17 past
Schedule::command('task')->everyTwoHours();
Schedule::command('task')->daily();
Schedule::command('task')->dailyAt('13:00');
Schedule::command('task')->twiceDaily(1, 13);  // 1am and 1pm
Schedule::command('task')->weekly();
Schedule::command('task')->weeklyOn(1, '8:00');  // Monday 8am
Schedule::command('task')->monthly();
Schedule::command('task')->monthlyOn(4, '15:00');  // 4th at 3pm
Schedule::command('task')->quarterly();
Schedule::command('task')->yearly();
Schedule::command('task')->yearlyOn(6, 1, '17:00');  // June 1st 5pm

// Custom cron
Schedule::command('task')->cron('0 * * * *');
```

## Timezone

```php
Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->dailyAt('09:00');
```

## Running the Scheduler

Add one cron entry to your server:

```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

For local development:

```bash
php artisan schedule:work  # Runs scheduler every minute
```

## Resources

- [Task Scheduling](https://laravel.com/docs/12.x/scheduling) — Official Laravel scheduling documentation

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*