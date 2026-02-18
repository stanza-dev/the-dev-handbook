---
source_course: "php-performance"
source_lesson: "php-performance-blackfire-profiling"
---

# Advanced Profiling with Blackfire

Blackfire is a production-grade profiler that provides detailed insights into PHP application performance without significant overhead.

## Why Use Blackfire Over Xdebug?

| Feature | Xdebug | Blackfire |
|---------|--------|----------|
| Production Use | No (high overhead) | Yes (minimal overhead) |
| Call Graph | Basic | Detailed with recommendations |
| Comparison | Manual | Built-in profile comparison |
| CI Integration | Limited | Full CI/CD integration |
| Memory Profiling | Basic | Detailed heap analysis |

## Installing Blackfire

```bash
# Install the probe (PHP extension)
curl -sSL https://packages.blackfire.io/gpg.key | sudo apt-key add -
echo "deb http://packages.blackfire.io/debian any main" | sudo tee /etc/apt/sources.list.d/blackfire.list
sudo apt-get update
sudo apt-get install blackfire-php

# Configure credentials
blackfire config --client-id=xxx --client-token=xxx
```

## Profiling a Script

```bash
# Profile a CLI script
blackfire run php my-script.php

# Profile with specific scenario
blackfire --samples=10 run php heavy-computation.php
```

## Profiling HTTP Requests

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

## Analyzing Results

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

## Assertions for CI/CD

```yaml
# .blackfire.yaml
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

## Comparison Profiling

```bash
# Create a reference profile
blackfire --reference run php benchmark.php

# Compare against reference
blackfire --reference=1 run php benchmark.php

# Output shows:
# - Wall Time: +15% (regression)
# - Memory: -5% (improvement)
# - SQL Queries: same
```

## Resources

- [Blackfire Documentation](https://blackfire.io/docs/introduction) — Official Blackfire profiler documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*