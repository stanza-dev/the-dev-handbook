---
source_course: "php-performance"
source_lesson: "php-performance-string-optimization"
---

# String & Memory Optimization

Strings and memory management can significantly impact PHP performance. Learn patterns that reduce memory usage and speed up string operations.

## String Concatenation

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

## Efficient String Functions

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

## Memory-Efficient File Processing

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

## Reducing Memory with References

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

## WeakMap for Cache (PHP 8+)

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

## Resources

- [WeakMap](https://www.php.net/manual/en/class.weakmap.php) — PHP WeakMap for memory-efficient caching

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*