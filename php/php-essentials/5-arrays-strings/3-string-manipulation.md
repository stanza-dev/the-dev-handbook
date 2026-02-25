---
source_course: "php-essentials"
source_lesson: "php-essentials-string-manipulation"
---

# String Manipulation

## Introduction

Strings are one of the most frequently used data types in any programming language, and PHP has an exceptionally rich set of string functions. From validating user input to generating URLs and formatting output, string manipulation is a skill you will use in every PHP project. This lesson covers string creation, searching, extracting, replacing, and formatting.

## Key Concepts

- **Single-quoted strings**: Literal strings with no variable interpolation.
- **Double-quoted strings**: Strings that expand variables and escape sequences.
- **Heredoc / Nowdoc**: Multi-line string syntaxes for embedding larger blocks of text.
- **String functions**: Built-in functions like `strlen()`, `strpos()`, `substr()`, `str_replace()`, and `explode()` for common operations.
- **PHP 8 helpers**: `str_contains()`, `str_starts_with()`, and `str_ends_with()` for readable membership checks.

## Real World Context

In production PHP code, you constantly work with strings: building SQL queries (safely, with prepared statements), constructing URLs, generating slugs from titles, parsing CSV data, formatting prices for display, and sanitizing user input. The string functions you learn here are the same ones powering every major PHP framework.

## Deep Dive

### String Creation

PHP offers four ways to create strings. Single and double quotes are the most common:

```php
<?php
// Single quotes - literal
$literal = 'Hello, $name';  // Outputs: Hello, $name

// Double quotes - interpolation
$name = 'World';
$greeting = "Hello, $name";  // Outputs: Hello, World

// Curly braces for complex expressions
$greeting = "Hello, {$user['name']}";

// Heredoc - for long strings with interpolation
$html = <<<HTML
<div class="container">
    <h1>Welcome, $name</h1>
</div>
HTML;

// Nowdoc - like single quotes, no interpolation
$raw = <<<'TEXT'
No $interpolation here
TEXT;
```

Use single quotes when you do not need interpolation — they are marginally faster and make intent clear.

### Common String Functions

These functions handle the most frequent string operations:

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

`trim()` is essential when processing user input from forms — users often accidentally include leading or trailing spaces.

### Searching and Finding

PHP provides both classic and modern search functions:

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

The PHP 8 functions (`str_contains`, `str_starts_with`, `str_ends_with`) are far more readable than the old `strpos() !== false` pattern.

### Extracting and Replacing

`substr()` extracts portions, and `str_replace()` swaps substrings:

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

Negative offsets in `substr()` count from the end of the string, which is handy for extracting file extensions or suffixes.

### Splitting and Joining

`explode()` splits a string into an array, and `implode()` joins an array into a string:

```php
<?php
$str = "apple,banana,cherry";

explode(',', $str);  // ['apple', 'banana', 'cherry']

$arr = ['a', 'b', 'c'];
implode('-', $arr);  // "a-b-c"
join('-', $arr);     // Same as implode

str_split('Hello', 2);  // ['He', 'll', 'o']
```

These two functions are the inverse of each other and are used constantly for CSV parsing, URL path manipulation, and tag processing.

### Formatting

`sprintf()` and `number_format()` produce formatted output:

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

`sprintf()` is preferable to string concatenation when building complex strings with multiple variables.

## Common Pitfalls

1. **Using `strpos()` without strict comparison** — `strpos('apple', 'a')` returns `0`, which is falsy. Always use `strpos($str, $needle) !== false` or switch to `str_contains()` on PHP 8+.
2. **Mixing up `str_replace()` argument order** — The order is `str_replace($search, $replace, $subject)`. A common mistake is passing the subject string as the first argument.
3. **Forgetting `strlen()` counts bytes, not characters** — For multibyte strings (UTF-8), use `mb_strlen()` instead. `strlen('é')` returns 2, but `mb_strlen('é')` returns 1.

## Best Practices

1. **Use PHP 8 string functions when available** — `str_contains()`, `str_starts_with()`, and `str_ends_with()` are more readable and less error-prone than `strpos()` comparisons.
2. **Use `mb_*` functions for international text** — If your application handles non-ASCII text, always use the `mb_` prefixed functions (`mb_strlen`, `mb_substr`, `mb_strtolower`) to handle multibyte characters correctly.
3. **Prefer `sprintf()` for complex formatting** — It is more readable than concatenation chains and makes the output format explicit.

## Summary

- Single-quoted strings are literal; double-quoted strings support variable interpolation.
- `strpos()` finds substring positions; PHP 8's `str_contains()` is a cleaner alternative.
- `substr()` extracts, `str_replace()` swaps, `explode()` splits, and `implode()` joins.
- `sprintf()` formats strings with placeholders; `number_format()` handles numeric display.
- Always use `mb_*` functions when working with multibyte (UTF-8) text.

## Code Examples

**URL slug generator combining strtolower, str_replace, preg_replace, and trim to transform titles into URL-safe strings**

```php
<?php
// URL slug generator using string functions
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