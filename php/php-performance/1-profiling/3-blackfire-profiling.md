---
source_course: "php-performance"
source_lesson: "php-performance-blackfire-profiling"
---

# Advanced Profiling with Blackfire

## Introduction
Blackfire is a production-grade profiler that provides detailed insights into PHP application performance without significant overhead.

## Key Concepts
- **Blackfire Profiler**: A production-safe profiling tool that samples code execution with minimal overhead.
- **Call Graph**: Visual representation of function calls, execution time, and memory allocation per function.
- **Assertions**: Blackfire tests that define performance budgets (e.g., "this endpoint must execute fewer than 10 SQL queries").
- **Continuous Profiling**: Periodic sampling in production to detect performance regressions over time.

## Real World Context
Blackfire is used by companies like Symfony, Platform.sh, and many enterprise PHP applications to profile in production without impacting users. Unlike Xdebug, it uses instrumentation that adds negligible overhead, making it safe for always-on production monitoring.

## Deep Dive
### Intro

Blackfire is a production-grade profiler that provides detailed insights into PHP application performance without significant overhead.

### Why use blackfire over xdebug?

| Feature | Xdebug | Blackfire |
|---------|--------|----------|
| Production Use | No (high overhead) | Yes (minimal overhead) |
| Call Graph | Basic | Detailed with recommendations |
| Comparison | Manual | Built-in profile comparison |
| CI Integration | Limited | Full CI/CD integration |
| Memory Profiling | Basic | Detailed heap analysis |

### Installing blackfire

```bash
curl -sSL https://packages.blackfire.io/gpg.key | sudo apt-key add -
echo "deb http://packages.blackfire.io/debian any main" | sudo tee /etc/apt/sources.list.d/blackfire.list
sudo apt-get update
sudo apt-get install blackfire-php

blackfire config --client-id=xxx --client-token=xxx
```

### Profiling a script

```bash
blackfire run php my-script.php

blackfire --samples=10 run php heavy-computation.php
```

### Profiling http requests

```php
<?php
// Install Blackfire SDK
composer require blackfire/php-sdk

use Blackfire\Client;
use Blackfire\Profile\Configuration;

$blackfire = new Client();

$config = new Configuration();
$config->setTitle('Homepage Profile');
$config->setSamples(10);

$probe = $blackfire->createProbe($config);

// Your code to profile
$result = processHeavyOperation();

$profile = $blackfire->endProbe($probe);
echo "Profile URL: " . $profile->getUrl();
```

### Analyzing results

Blackfire provides:

```php
<?php
// Key metrics to analyze
$metrics = [
    'wall_time' => 'Total execution time',
    'cpu_time' => 'CPU processing time',
    'io_time' => 'Time waiting for I/O',
    'memory' => 'Peak memory usage',
    'network_in' => 'Data received',
    'network_out' => 'Data sent',
    'sql_queries' => 'Number of database queries',
    'http_requests' => 'External HTTP calls',
];
```

### Assertions for ci/cd

```yaml
scenarios:
  Homepage:
    - path: /
      assertions:
        - main.wall_time < 200ms
        - main.peak_memory < 50mb
        - metrics.sql.queries.count < 10
        
  API Endpoint:
    - path: /api/users
      assertions:
        - main.wall_time < 100ms
        - metrics.http.requests.count == 0
```

### Comparison profiling

```bash
blackfire --reference run php benchmark.php

blackfire --reference=1 run php benchmark.php

```

## Common Pitfalls
1. **Only profiling on-demand** — Some performance issues only appear under load or with specific data patterns. Use continuous profiling to catch regressions early.
2. **Ignoring memory profiles** — CPU time gets all the attention, but memory allocation can be the real bottleneck, especially in queue workers and batch processes.

## Best Practices
1. **Set up performance assertions in CI** — Define budgets like "max 15 queries per page" and "response under 200ms" that fail the build if violated.
2. **Profile queue workers and CLI commands** — Web requests get profiled naturally, but long-running workers often have memory leaks and inefficient loops that go unnoticed.

## Summary
- Blackfire provides production-safe profiling with minimal overhead.
- Use performance assertions in CI/CD to catch regressions before deployment.
- Profile both web requests and background workers for complete coverage.

## Resources

- [Blackfire Documentation](https://blackfire.io/docs/introduction) — Official Blackfire profiler documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*