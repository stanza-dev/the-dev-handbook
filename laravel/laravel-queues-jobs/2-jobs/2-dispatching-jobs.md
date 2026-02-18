---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-dispatching-jobs"
---

# Dispatching Jobs

Once you've created a job, you need to dispatch it to the queue for processing.

## Basic Dispatching

```php
use App\Jobs\ProcessPodcast;

// In a controller
public function store(Request $request)
{
    $podcast = Podcast::create($request->validated());

    // Dispatch to queue
    ProcessPodcast::dispatch($podcast);

    return redirect()->route('podcasts.index')
        ->with('status', 'Podcast is being processed!');
}
```

## Dispatch Methods

```php
// Standard dispatch (queued)
ProcessPodcast::dispatch($podcast);

// Dispatch if condition is true
ProcessPodcast::dispatchIf($shouldProcess, $podcast);

// Dispatch unless condition is true
ProcessPodcast::dispatchUnless($skipProcessing, $podcast);

// Dispatch synchronously (bypasses queue)
ProcessPodcast::dispatchSync($podcast);

// Dispatch after response is sent
ProcessPodcast::dispatchAfterResponse($podcast);
```

## Delayed Dispatching

```php
// Delay by minutes
ProcessPodcast::dispatch($podcast)
    ->delay(now()->addMinutes(10));

// Delay by specific time
SendReminder::dispatch($user)
    ->delay(now()->addHours(24));

// Delay until specific datetime
PublishPost::dispatch($post)
    ->delay($post->scheduled_at);
```

## Specifying Queue and Connection

```php
// Dispatch to specific queue
ProcessPodcast::dispatch($podcast)
    ->onQueue('processing');

// Dispatch to specific connection
ProcessPodcast::dispatch($podcast)
    ->onConnection('redis');

// Both
ProcessPodcast::dispatch($podcast)
    ->onConnection('redis')
    ->onQueue('high');
```

## Chain Jobs

Run jobs in sequence:

```php
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast($podcast),
    new OptimizeAudio($podcast),
    new PublishPodcast($podcast),
    new NotifySubscribers($podcast),
])->dispatch();

// With error handling
Bus::chain([
    new ProcessPodcast($podcast),
    new PublishPodcast($podcast),
])->catch(function (Throwable $e) {
    // Handle chain failure
    Log::error('Podcast chain failed', ['error' => $e->getMessage()]);
})->dispatch();
```

## Job Batching

Process multiple jobs as a batch:

```php
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([
    new ImportCsv($file1),
    new ImportCsv($file2),
    new ImportCsv($file3),
])->then(function (Batch $batch) {
    // All jobs completed successfully
    Log::info('Import completed!');
})->catch(function (Batch $batch, Throwable $e) {
    // First failure detected
    Log::error('Import failed', ['error' => $e->getMessage()]);
})->finally(function (Batch $batch) {
    // Batch finished (success or failure)
})->name('CSV Import')
  ->allowFailures()
  ->dispatch();

// Get batch ID for tracking
$batchId = $batch->id;
```

### Checking Batch Status

```php
use Illuminate\Support\Facades\Bus;

$batch = Bus::findBatch($batchId);

$batch->id;               // Batch ID
$batch->name;             // 'CSV Import'
$batch->totalJobs;        // Total jobs in batch
$batch->pendingJobs;      // Jobs waiting
$batch->processedJobs();  // Jobs completed
$batch->failedJobs;       // Jobs failed
$batch->progress();       // Percentage complete
$batch->finished();       // Is batch done?
$batch->cancelled();      // Was batch cancelled?
```

### Adding Jobs to Batch

```php
// Inside a batched job
public function handle(): void
{
    if ($this->batch()->cancelled()) {
        return;
    }

    // Add more jobs to the batch
    $this->batch()->add([
        new ProcessChunk($this->chunk),
    ]);
}
```

## Dispatching from Controllers

```php
class PodcastController extends Controller
{
    public function store(StorePodcastRequest $request)
    {
        $podcast = Podcast::create($request->validated());

        ProcessPodcast::dispatch($podcast);

        return response()->json([
            'message' => 'Podcast uploaded and queued for processing',
            'podcast' => $podcast,
        ], 201);
    }

    public function publish(Podcast $podcast)
    {
        Bus::chain([
            new OptimizePodcast($podcast),
            new GenerateThumbnail($podcast),
            new PublishPodcast($podcast),
        ])->dispatch();

        return back()->with('status', 'Publishing in progress...');
    }
}
```

## Resources

- [Dispatching Jobs](https://laravel.com/docs/12.x/queues#dispatching-jobs) — Official documentation on dispatching jobs

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*