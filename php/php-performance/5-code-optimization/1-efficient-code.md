---
source_course: "php-performance"
source_lesson: "php-performance-efficient-code"
---

# Writing Efficient PHP Code

Small optimizations add up. Write efficient code from the start.

## String Operations

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

## Array Operations

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

## Loops

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

## Memory Efficiency

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

## Type Declarations

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

## Precomputation

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

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*