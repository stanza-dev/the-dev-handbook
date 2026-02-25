---
source_course: "php-performance"
source_lesson: "php-performance-profiling-basics"
---

# Measuring PHP Performance

## Introduction
Before optimizing, measure. Profile your code to find actual bottlenecks.

## Key Concepts
- **microtime(true)**: Returns current time as a float with microsecond precision for measuring code execution.
- **memory_get_peak_usage()**: Reports the maximum memory allocated during script execution.
- **Xdebug Profiler**: Generates cachegrind files showing function-level CPU time and call counts.
- **Baseline Measurement**: Always measure before optimizing — premature optimization wastes effort on non-bottlenecks.

## Real World Context
Netflix, Facebook, and Shopify all invest heavily in PHP/HHVM performance monitoring. A 100ms increase in page load time can reduce conversion rates by 7% (Akamai study). Profiling identifies the specific functions consuming the most time and memory, preventing wasted effort on micro-optimizations that don't matter.

## Deep Dive
### Intro

Before optimizing, measure. Profile your code to find actual bottlenecks.

### Basic timing

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

### Memory usage

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

### Benchmark class

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

### Xdebug profiling

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

### Key metrics

| Metric | What It Measures | Tool |
|--------|-----------------|------|
| Response Time | Total request duration | Timer, APM |
| Memory Usage | RAM consumption | memory_get_usage() |
| CPU Time | Processing time | Xdebug, Blackfire |
| I/O Wait | Database, file, network | APM, profilers |
| Throughput | Requests per second | Load testing |

## Common Pitfalls
1. **Optimizing without profiling** — Developers often guess where bottlenecks are and get it wrong. Always measure first with actual profiling data.
2. **Profiling in development only** — Development environments differ from production (different data sizes, no load, different hardware). Use sampling profilers like Blackfire in production.

## Best Practices
1. **Establish performance baselines** — Record response times, memory usage, and throughput before making changes so you can measure improvement.
2. **Profile with realistic data** — Use production-like datasets for profiling. A query that's fast with 100 rows may be catastrophic with 1 million.

## Summary

> **PHP 8.5 Note:** Fatal errors now include a full backtrace, making it easier to diagnose performance-related crashes like out-of-memory errors and timeouts without additional tooling.
- Always profile before optimizing — use `microtime(true)` for timing and `memory_get_peak_usage()` for memory.
- Use Xdebug or Blackfire for detailed function-level profiling.
- Establish baselines and profile with production-like data to find real bottlenecks.

## Resources

- [Xdebug Profiling](https://xdebug.org/docs/profiler) — Xdebug profiler documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*