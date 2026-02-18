---
source_course: "php-essentials"
source_lesson: "php-essentials-string-manipulation"
---

# String Manipulation

Strings are one of the most common data types. PHP offers powerful functions for string manipulation.

## String Creation

```php
<?php
// Single quotes - literal
$literal = 'Hello, $name';  // Outputs: Hello, $name

// Double quotes - interpolation
$name = 'World';
$greeting = "Hello, $name";  // Outputs: Hello, World

// Curly braces for complex expressions
$greeting = "Hello, {$user['name']}";

// Heredoc - for long strings
$html = <<<HTML
<div class="container">
    <h1>Welcome, $name</h1>
</div>
HTML;

// Nowdoc - like single quotes
$raw = <<<'TEXT'
No $interpolation here
TEXT;
```

## Common String Functions

```php
<?php
$str = "Hello, World!";

strlen($str);           // 13 - length
strtoupper($str);       // HELLO, WORLD!
strtolower($str);       // hello, world!
ucfirst($str);          // Hello, world!
ucwords("hello world"); // Hello World

trim("  hello  ");      // "hello" - remove whitespace
ltrim("  hello");       // "hello" - left trim
rtrim("hello  ");       // "hello" - right trim
```

## Searching & Finding

```php
<?php
$str = "Hello, World!";

strpos($str, 'World');      // 7 - position (case-sensitive)
stripos($str, 'world');     // 7 - case-insensitive
strrpos($str, 'o');         // 8 - last occurrence

str_contains($str, 'World'); // true (PHP 8+)
str_starts_with($str, 'Hello'); // true (PHP 8+)
str_ends_with($str, '!');    // true (PHP 8+)
```

## Extracting & Replacing

```php
<?php
$str = "Hello, World!";

substr($str, 0, 5);         // "Hello"
substr($str, 7);            // "World!"
substr($str, -6);           // "World!"

str_replace('World', 'PHP', $str);  // "Hello, PHP!"
str_ireplace('WORLD', 'PHP', $str); // Case-insensitive

// Replace multiple
str_replace(
    ['Hello', 'World'],
    ['Hi', 'PHP'],
    $str
);  // "Hi, PHP!"
```

## Splitting & Joining

```php
<?php
$str = "apple,banana,cherry";

explode(',', $str);  // ['apple', 'banana', 'cherry']

$arr = ['a', 'b', 'c'];
implode('-', $arr);  // "a-b-c"
join('-', $arr);     // Same as implode

str_split('Hello', 2);  // ['He', 'll', 'o']
```

## Formatting

```php
<?php
// Printf-style formatting
sprintf("Hello, %s!", "World");  // "Hello, World!"
sprintf("%05d", 42);              // "00042"
sprintf("%.2f", 3.14159);         // "3.14"

// Number formatting
number_format(1234567.891, 2);    // "1,234,567.89"
number_format(1234567.891, 2, ',', ' '); // "1 234 567,89"
```

## Code Examples

**URL slug generator using string functions**

```php
<?php
// URL slug generator
function createSlug(string $title): string {
    // Convert to lowercase
    $slug = strtolower($title);
    
    // Replace spaces with hyphens
    $slug = str_replace(' ', '-', $slug);
    
    // Remove special characters (keep only letters, numbers, hyphens)
    $slug = preg_replace('/[^a-z0-9-]/', '', $slug);
    
    // Remove multiple consecutive hyphens
    $slug = preg_replace('/-+/', '-', $slug);
    
    // Trim hyphens from ends
    return trim($slug, '-');
}

$title = "Hello World! This is PHP 8.4";
echo createSlug($title);
// Output: hello-world-this-is-php-84
?>
```


## Resources

- [String Functions](https://www.php.net/manual/en/ref.strings.php) — Complete reference of PHP string functions

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*