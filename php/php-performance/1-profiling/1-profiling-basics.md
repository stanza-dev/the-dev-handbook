---
source_course: "php-performance"
source_lesson: "php-performance-profiling-basics"
---

# Measuring PHP Performance

Before optimizing, measure. Profile your code to find actual bottlenecks.

## Basic Timing

```php
<?php
$start = microtime(true);

// Code to measure
for ($i = 0; $i < 10000; $i++) {
    $result = someFunction($i);
}

$end = microtime(true);
$duration = ($end - $start) * 1000;  // Convert to milliseconds

echo "Execution time: {$duration}ms\n";
```

## Memory Usage

```php
<?php
$memStart = memory_get_usage();

// Code that allocates memory
$data = range(1, 100000);

$memEnd = memory_get_usage();
$memUsed = ($memEnd - $memStart) / 1024 / 1024;  // MB

echo "Memory used: {$memUsed}MB\n";
echo "Peak memory: " . (memory_get_peak_usage() / 1024 / 1024) . "MB\n";
```

## Benchmark Class

```php
<?php
class Benchmark
{
    private float $startTime;
    private int $startMemory;
    private array $markers = [];
    
    public function start(): void
    {
        $this->startTime = microtime(true);
        $this->startMemory = memory_get_usage();
    }
    
    public function mark(string $name): void
    {
        $this->markers[$name] = [
            'time' => microtime(true) - $this->startTime,
            'memory' => memory_get_usage() - $this->startMemory,
        ];
    }
    
    public function report(): array
    {
        return [
            'total_time_ms' => (microtime(true) - $this->startTime) * 1000,
            'total_memory_mb' => (memory_get_usage() - $this->startMemory) / 1024 / 1024,
            'peak_memory_mb' => memory_get_peak_usage() / 1024 / 1024,
            'markers' => $this->markers,
        ];
    }
}

// Usage
$bench = new Benchmark();
$bench->start();

$users = fetchUsers();
$bench->mark('fetch_users');

$processed = processUsers($users);
$bench->mark('process_users');

print_r($bench->report());
```

## Xdebug Profiling

```ini
; php.ini
xdebug.mode=profile
xdebug.output_dir=/tmp/xdebug
xdebug.profiler_output_name=cachegrind.out.%p
```

```php
<?php
// Trigger profiling for specific code
if (function_exists('xdebug_start_trace')) {
    xdebug_start_trace('/tmp/trace');
}

// Your code here

if (function_exists('xdebug_stop_trace')) {
    xdebug_stop_trace();
}
```

## Key Metrics

| Metric | What It Measures | Tool |
|--------|-----------------|------|
| Response Time | Total request duration | Timer, APM |
| Memory Usage | RAM consumption | memory_get_usage() |
| CPU Time | Processing time | Xdebug, Blackfire |
| I/O Wait | Database, file, network | APM, profilers |
| Throughput | Requests per second | Load testing |

## Resources

- [Xdebug Profiling](https://xdebug.org/docs/profiler) — Xdebug profiler documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*