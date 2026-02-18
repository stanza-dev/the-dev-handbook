---
source_course: "php-performance"
source_lesson: "php-performance-preloading-advanced"
---

# Advanced Preloading Strategies

Preloading (PHP 7.4+) loads PHP files into OPcache at server startup, making them permanently available without recompilation.

## How Preloading Works

```
Server Start → Execute preload.php → Classes in shared memory
                                              ↓
              All requests share preloaded classes (zero cost)
```

## Basic Preload Script

```php
<?php
// preload.php

// Option 1: Preload specific files
$files = [
    __DIR__ . '/vendor/autoload.php',
    __DIR__ . '/src/Kernel.php',
    __DIR__ . '/src/Entity/User.php',
    __DIR__ . '/src/Entity/Order.php',
];

foreach ($files as $file) {
    require_once $file;
}
```

## Automated Class Discovery

```php
<?php
// preload.php - Discover and preload hot classes

class Preloader
{
    private array $loaded = [];
    private array $ignored = [];
    
    public function __construct(
        private array $paths,
        private array $ignorePatterns = []
    ) {}
    
    public function load(): int
    {
        foreach ($this->paths as $path) {
            $this->loadPath($path);
        }
        
        return count($this->loaded);
    }
    
    private function loadPath(string $path): void
    {
        if (is_file($path)) {
            $this->loadFile($path);
            return;
        }
        
        $iterator = new RecursiveIteratorIterator(
            new RecursiveDirectoryIterator($path)
        );
        
        foreach ($iterator as $file) {
            if ($file->isFile() && $file->getExtension() === 'php') {
                $this->loadFile($file->getPathname());
            }
        }
    }
    
    private function loadFile(string $path): void
    {
        foreach ($this->ignorePatterns as $pattern) {
            if (preg_match($pattern, $path)) {
                $this->ignored[] = $path;
                return;
            }
        }
        
        try {
            require_once $path;
            $this->loaded[] = $path;
        } catch (Throwable $e) {
            // Log but don't fail - some files may have dependencies
        }
    }
    
    public function getLoaded(): array
    {
        return $this->loaded;
    }
}

// Usage
$preloader = new Preloader(
    paths: [
        __DIR__ . '/src/Entity',
        __DIR__ . '/src/Service',
        __DIR__ . '/src/Repository',
    ],
    ignorePatterns: [
        '/Test\.php$/',
        '/Fixture/',
    ]
);

$count = $preloader->load();
error_log("Preloaded $count files");
```

## Configuration

```ini
; php.ini
opcache.preload=/var/www/app/preload.php
opcache.preload_user=www-data

; Ensure OPcache is enabled
opcache.enable=1
opcache.enable_cli=0
```

## What to Preload

**Good candidates:**
- Framework core classes
- Heavily used entities/models
- Service classes
- Frequently used utilities

**Avoid preloading:**
- Test files
- CLI-only commands
- Rarely used exception classes
- Development tools

## Measuring Impact

```php
<?php
// Check preloaded classes
$status = opcache_get_status();
$preloadStats = $status['preload_statistics'] ?? [];

echo "Preloaded functions: " . ($preloadStats['functions'] ?? 0) . "\n";
echo "Preloaded classes: " . ($preloadStats['classes'] ?? 0) . "\n";
echo "Memory used: " . ($preloadStats['memory_consumption'] ?? 0) . " bytes\n";
```

## Caveats

1. **Server restart required** - Changes need FPM/server restart
2. **No hot reload** - Development may need preloading disabled
3. **Dependency order** - Files load in order; dependencies must load first
4. **Memory consumption** - Preloaded classes use memory even if unused

## Resources

- [OPcache Preloading](https://www.php.net/manual/en/opcache.preloading.php) — PHP preloading documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*