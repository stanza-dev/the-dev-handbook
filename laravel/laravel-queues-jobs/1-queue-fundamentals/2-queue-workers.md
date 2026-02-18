---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-queue-workers"
---

# Configuring Queue Workers

Queue workers are long-running processes that continuously process jobs from the queue. Proper configuration is essential for production applications.

## Worker Commands

```bash
# Basic worker
php artisan queue:work

# Specific connection and queue
php artisan queue:work redis --queue=emails

# Common options
php artisan queue:work \
    --tries=3 \           # Retry failed jobs 3 times
    --timeout=60 \         # Max seconds per job
    --memory=128 \         # Restart if memory exceeds 128MB
    --sleep=3 \            # Seconds to sleep when no jobs
    --max-jobs=1000 \      # Restart after processing 1000 jobs
    --max-time=3600        # Restart after 1 hour
```

## Worker vs Listen

```bash
# queue:work - Keeps framework booted (faster)
php artisan queue:work

# queue:listen - Reboots for each job (good for dev)
php artisan queue:listen
```

| Mode | Performance | Code Changes |
|------|-------------|---------------|
| `work` | Faster | Needs restart to see changes |
| `listen` | Slower | Auto-detects code changes |

## Queue Priorities

Process high-priority jobs first:

```bash
php artisan queue:work --queue=high,default,low
```

Dispatch to specific queues:

```php
// Dispatch to high priority queue
SendUrgentEmail::dispatch($user)->onQueue('high');

// Normal priority (default queue)
ProcessReport::dispatch($data);

// Low priority
CleanupOldFiles::dispatch()->onQueue('low');
```

## Supervisor Configuration

For production, use Supervisor to keep workers running:

```ini
# /etc/supervisor/conf.d/laravel-worker.conf
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/html/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=8
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/worker.log
stopwaitsecs=3600
```

```bash
# Apply configuration
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start "laravel-worker:*"
```

## Restarting Workers

After deploying new code:

```bash
# Graceful restart (finishes current job)
php artisan queue:restart
```

**Important**: Workers cache your application. Always restart after deployments!

## Handling Failed Jobs

```bash
# View failed jobs
php artisan queue:failed

# Retry a specific job
php artisan queue:retry 5

# Retry all failed jobs
php artisan queue:retry all

# Delete a failed job
php artisan queue:forget 5

# Delete all failed jobs
php artisan queue:flush
```

Failed jobs are stored in the `failed_jobs` table:

```php
Schema::create('failed_jobs', function (Blueprint $table) {
    $table->id();
    $table->string('uuid')->unique();
    $table->text('connection');
    $table->text('queue');
    $table->longText('payload');
    $table->longText('exception');
    $table->timestamp('failed_at')->useCurrent();
});
```

## Monitoring with Horizon

For Redis queues, Laravel Horizon provides a beautiful dashboard:

```bash
composer require laravel/horizon
php artisan horizon:install
php artisan migrate
```

Access dashboard at `/horizon`.

## Worker Health Checks

```php
// In a scheduled command
$schedule->command('queue:monitor redis:default,redis:emails --max=100')
    ->everyMinute();
```

Alert when queues have too many jobs waiting.

## Resources

- [Running Queue Workers](https://laravel.com/docs/12.x/queues#running-the-queue-worker) — Official documentation on queue workers

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*