---
source_course: "php-performance"
source_lesson: "php-performance-efficient-code"
---

# Writing Efficient PHP Code

## Introduction
Small optimizations add up. Write efficient code from the start.

## Key Concepts
- **Generators**: `yield` keyword creates iterators that process one item at a time, using constant memory regardless of dataset size.
- **array_flip() for Lookups**: Convert value-search (`in_array()`, O(n)) to key-search (`isset()`, O(1)) by flipping the array.
- **Early Return**: Exiting functions early when conditions are met, avoiding unnecessary computation.
- **String vs Array Functions**: Native PHP functions like `isset()`, `count()`, and `implode()` are C-optimized and vastly faster than userland equivalents.

## Real World Context
PHP 8.x's JIT compiler makes native function calls even faster, but algorithmic improvements always outweigh micro-optimizations. Replacing a single `in_array()` inside a loop with `isset()` on a flipped array can turn an O(n²) operation into O(n), making the difference between a 30-second and 30-millisecond batch job.

## Deep Dive
### Intro

Small optimizations add up. Write efficient code from the start.

### String operations

```php
<?php
// BAD: Concatenation in loop creates many strings
$result = '';
for ($i = 0; $i < 10000; $i++) {
    $result .= "Line $i\n";
}

// GOOD: Build array, then join
$lines = [];
for ($i = 0; $i < 10000; $i++) {
    $lines[] = "Line $i";
}
$result = implode("\n", $lines);

// BETTER: Use output buffering
ob_start();
for ($i = 0; $i < 10000; $i++) {
    echo "Line $i\n";
}
$result = ob_get_clean();
```

### Array operations

```php
<?php
// isset() vs array_key_exists()
// isset() is faster but returns false for null values
if (isset($array[$key])) {}  // Fast
if (array_key_exists($key, $array)) {}  // Slower but checks null

// in_array() complexity is O(n)
// For large arrays, use array_flip + isset
$values = ['a', 'b', 'c', ...];
if (in_array($needle, $values)) {}  // O(n)

$flipped = array_flip($values);
if (isset($flipped[$needle])) {}  // O(1)

// array_map vs foreach
$squared = array_map(fn($x) => $x * $x, $numbers);  // Creates new array

foreach ($numbers as &$n) {  // Modifies in place
    $n = $n * $n;
}
```

### Loops

```php
<?php
// BAD: count() called every iteration
for ($i = 0; $i < count($array); $i++) {}

// GOOD: Cache count
$count = count($array);
for ($i = 0; $i < $count; $i++) {}

// Better: foreach for arrays
foreach ($array as $item) {}

// Avoid function calls in conditions
while (strlen($string) > 0) {}  // BAD
while ($string !== '') {}  // GOOD
```

### Memory efficiency

```php
<?php
// Use generators for large datasets
function readLargeFile(string $path): Generator {
    $handle = fopen($path, 'r');
    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
    fclose($handle);
}

// Only one line in memory at a time
foreach (readLargeFile('huge.csv') as $line) {
    processLine($line);
}

// Unset large variables when done
$largeData = fetchHugeDataset();
processData($largeData);
unset($largeData);  // Free memory immediately
```

### Type declarations

```php
<?php
declare(strict_types=1);

// Typed code can be optimized better
function calculateTotal(array $items): float {
    $total = 0.0;
    foreach ($items as $item) {
        $total += $item['price'] * $item['quantity'];
    }
    return $total;
}
```

### Precomputation

```php
<?php
// BAD: Calculate on every call
function getDayName(int $day): string {
    return match($day) {
        0 => 'Sunday',
        1 => 'Monday',
        // ...
    };
}

// GOOD: Use constant array
const DAYS = ['Sunday', 'Monday', 'Tuesday', ...];

function getDayName(int $day): string {
    return DAYS[$day];
}
```

## Common Pitfalls
1. **Using `in_array()` in loops** — `in_array()` is O(n) per call. Inside a loop over m items, it becomes O(n×m). Use `array_flip()` + `isset()` for O(1) lookups.
2. **Loading entire files into memory** — `file_get_contents()` on a 500MB CSV file crashes the process. Use generators or `SplFileObject` for line-by-line processing.

## Best Practices
1. **Use generators for large datasets** — Process CSV files, database result sets, and API pagination with generators to maintain constant memory usage.
2. **Prefer native PHP functions over loops** — `array_map()`, `array_filter()`, `array_column()`, and `implode()` are implemented in C and are faster than equivalent foreach loops.

## Summary
- Use generators (`yield`) for processing large datasets with constant memory usage.
- Replace `in_array()` with `array_flip()` + `isset()` for O(1) lookups in loops.
- Prefer native PHP array functions over manual loops for better performance.

## Resources

- [PHP Performance Tips](https://www.php.net/manual/en/intro.opcache.php) — PHP OPcache introduction and performance optimization

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*