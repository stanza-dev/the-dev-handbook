---
source_course: "php-performance"
source_lesson: "php-performance-string-optimization"
---

# String & Memory Optimization

## Introduction
Strings and memory management can significantly impact PHP performance. Learn patterns that reduce memory usage and speed up string operations.

## Key Concepts
- **str_starts_with() / str_ends_with()**: PHP 8.0+ native functions replacing `strpos() === 0` and `substr()` comparisons — clearer and faster.
- **WeakMap**: PHP 8.0+ data structure where entries are automatically garbage collected when their key objects are destroyed.
- **String Interpolation vs Concatenation**: Double-quoted strings with variables are marginally faster than concatenation but the difference is negligible.
- **mb_string Functions**: Required for multi-byte (UTF-8) string operations — `strlen()` counts bytes, `mb_strlen()` counts characters.

## Real World Context
String operations are among the most frequent in PHP applications — URL processing, template rendering, and data transformation all involve heavy string manipulation. While individual string optimizations are small, they compound in tight loops processing thousands of records.

## Deep Dive
### Intro

Strings and memory management can significantly impact PHP performance. Learn patterns that reduce memory usage and speed up string operations.

### String concatenation

```php
<?php
// BAD: Creates many intermediate strings
$result = '';
for ($i = 0; $i < 10000; $i++) {
    $result = $result . "Line $i\n";  // Creates new string each time
}

// BETTER: Use .= operator (slightly optimized)
$result = '';
for ($i = 0; $i < 10000; $i++) {
    $result .= "Line $i\n";
}

// BEST: Collect in array, join once
$lines = [];
for ($i = 0; $i < 10000; $i++) {
    $lines[] = "Line $i";
}
$result = implode("\n", $lines);

// OR: Use output buffering
ob_start();
for ($i = 0; $i < 10000; $i++) {
    echo "Line $i\n";
}
$result = ob_get_clean();
```

### Efficient string functions

```php
<?php
// Choose the right function for the job

// Checking if string starts with prefix
// BAD
if (substr($string, 0, 5) === 'Hello') {}

// GOOD (PHP 8+)
if (str_starts_with($string, 'Hello')) {}

// Checking if string contains substring
// BAD
if (strpos($string, 'needle') !== false) {}

// GOOD (PHP 8+)
if (str_contains($string, 'needle')) {}

// Case-insensitive comparison
// BAD
if (strtolower($a) === strtolower($b)) {}

// GOOD
if (strcasecmp($a, $b) === 0) {}
```

### Memory-efficient file processing

```php
<?php
// BAD: Loads entire file into memory
$content = file_get_contents('large.csv');
$lines = explode("\n", $content);
foreach ($lines as $line) {
    process($line);
}
// Memory: O(file_size)

// GOOD: Stream line by line
$handle = fopen('large.csv', 'r');
while (($line = fgets($handle)) !== false) {
    process($line);
}
fclose($handle);
// Memory: O(1) - only one line at a time

// EVEN BETTER: Use SplFileObject
$file = new SplFileObject('large.csv');
foreach ($file as $line) {
    process($line);
}
```

### Reducing memory with references

```php
<?php
// BAD: Copies array for each iteration
function processItems(array $items): void
{
    foreach ($items as $item) {  // No copy needed
        echo $item;
    }
}

// When modifying, use reference to avoid copy
function normalizeItems(array &$items): void
{
    foreach ($items as &$item) {
        $item = strtolower(trim($item));
    }
    unset($item);  // Break the reference!
}

// CAUTION: Unset the reference after foreach!
```

### Weakmap for cache (php 8+)

```php
<?php
// Regular array holds strong references (memory leak potential)
class Cache
{
    private array $data = [];  // Objects never freed!
    
    public function compute(object $key, callable $fn): mixed
    {
        $hash = spl_object_hash($key);
        return $this->data[$hash] ??= $fn();
    }
}

// WeakMap: entries auto-removed when key is garbage collected
class WeakCache
{
    private WeakMap $data;
    
    public function __construct()
    {
        $this->data = new WeakMap();
    }
    
    public function compute(object $key, callable $fn): mixed
    {
        return $this->data[$key] ??= $fn();
    }
}

// When $user is unset elsewhere, WeakMap entry auto-removes
$cache = new WeakCache();
$user = new User(1);
$cache->compute($user, fn() => expensiveCalculation());
unset($user);  // Cache entry freed automatically
```

## Common Pitfalls
1. **Using `strpos()` for prefix checks** — `strpos($str, 'prefix') === 0` is less readable and slightly slower than `str_starts_with($str, 'prefix')` (PHP 8.0+).
2. **Memory leaks from circular references** — Object caches using `SplObjectStorage` or arrays prevent garbage collection of referenced objects. Use `WeakMap` instead to allow automatic cleanup.

## Best Practices
1. **Use PHP 8.0+ string functions** — `str_starts_with()`, `str_ends_with()`, and `str_contains()` are more readable and optimized than their `strpos()`/`substr()` equivalents.
2. **Use WeakMap for object caches** — When caching computed values for objects, `WeakMap` entries are automatically removed when the key object is garbage collected, preventing memory leaks.

## Summary

> **PHP 8.5 Features:** `array_first()` and `array_last()` provide cleaner alternatives to `reset()`/`end()`. The pipe operator `|>` enables readable function chaining: `$result = $input |> trim(...) |> strtolower(...);`
- Use `str_starts_with()`, `str_ends_with()`, `str_contains()` (PHP 8.0+) instead of `strpos()` workarounds.
- Use `WeakMap` for object caches to prevent memory leaks from circular references.
- Choose `mb_*` functions when working with multi-byte character encodings.

## Resources

- [WeakMap](https://www.php.net/manual/en/class.weakmap.php) — PHP WeakMap for memory-efficient caching

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*