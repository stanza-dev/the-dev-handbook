---
source_course: "php-performance"
source_lesson: "php-performance-opcache-config"
---

# Configuring OPcache

OPcache stores precompiled bytecode in memory, eliminating the need to parse PHP files on every request.

## How OPcache Works

```
Without OPcache:
[Request] → [Read File] → [Parse] → [Compile] → [Execute]
                                     ↑
                              (Every request)

With OPcache:
[Request] → [Read from Cache] → [Execute]
                   ↑
            (Compiled once)
```

## Enable OPcache

```ini
; php.ini
zend_extension=opcache
opcache.enable=1
opcache.enable_cli=0  ; Enable for CLI scripts if needed
```

## Production Configuration

```ini
; Memory allocation
opcache.memory_consumption=256      ; MB of memory for cached scripts
opcache.interned_strings_buffer=32  ; MB for interned strings
opcache.max_accelerated_files=20000 ; Max files to cache

; Performance settings
opcache.validate_timestamps=0       ; Don't check file changes (production!)
opcache.revalidate_freq=0           ; Ignored when validate_timestamps=0

; Optimization
opcache.optimization_level=0x7FFFFFFF  ; All optimizations
opcache.save_comments=0                 ; Remove comments (if not using reflection)
opcache.enable_file_override=1          ; Faster file operations

; Security
opcache.restrict_api=''             ; Restrict API to specific paths
```

## Development Configuration

```ini
; Check file changes for immediate updates
opcache.validate_timestamps=1
opcache.revalidate_freq=2  ; Check every 2 seconds
```

## Preloading (PHP 7.4+)

```php
<?php
// preload.php
$files = [
    __DIR__ . '/vendor/autoload.php',
    __DIR__ . '/src/Entity/User.php',
    __DIR__ . '/src/Entity/Order.php',
    // Add frequently used classes
];

foreach ($files as $file) {
    opcache_compile_file($file);
}
```

```ini
; php.ini
opcache.preload=/var/www/app/preload.php
opcache.preload_user=www-data
```

## Monitoring OPcache

```php
<?php
function opcacheStats(): array
{
    if (!function_exists('opcache_get_status')) {
        return ['error' => 'OPcache not available'];
    }
    
    $status = opcache_get_status();
    
    return [
        'enabled' => $status['opcache_enabled'],
        'memory_used_mb' => $status['memory_usage']['used_memory'] / 1024 / 1024,
        'memory_free_mb' => $status['memory_usage']['free_memory'] / 1024 / 1024,
        'hit_rate' => $status['opcache_statistics']['opcache_hit_rate'],
        'scripts_cached' => $status['opcache_statistics']['num_cached_scripts'],
        'restarts' => $status['opcache_statistics']['oom_restarts'],
    ];
}
```

## Clearing OPcache

```php
<?php
// Clear all cache
opcache_reset();

// Clear specific file
opcache_invalidate('/path/to/file.php', true);

// Deployment script
if (opcache_get_status()) {
    opcache_reset();
    echo "OPcache cleared\n";
}
```

## Resources

- [OPcache Documentation](https://www.php.net/manual/en/book.opcache.php) — PHP OPcache documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*