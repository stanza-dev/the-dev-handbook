---
source_course: "php-essentials"
source_lesson: "php-essentials-first-php-script"
---

# Writing Your First PHP Script

## Introduction
Now that your environment is ready, it is time to write your first PHP program. This lesson covers PHP tags, the `echo` statement, embedding PHP in HTML, and comment styles — everything you need to start producing output.

## Key Concepts
- **PHP Tags (`<?php ... ?>`)**: Delimiters that tell the server where PHP code begins and ends.
- **`echo` Statement**: The primary way to output text and HTML from PHP.
- **Short Echo Syntax (`<?= ... ?>`)**: A shorthand for `<?php echo ... ?>`, commonly used inside HTML templates.
- **Case Sensitivity**: PHP keywords are case-insensitive, but variable names are case-sensitive.

## Real World Context
Every PHP application begins with these fundamentals. Whether you are rendering a dynamic web page, generating a JSON API response, or processing a form submission, you will use `echo`, PHP tags, and comments daily. Understanding how PHP mixes with HTML is essential for templating and for working with frameworks like Laravel's Blade.

## Deep Dive

### PHP Tags

PHP code must be enclosed in opening and closing tags:

```php
<?php
// Your PHP code goes here
?>
```

The closing `?>` tag is optional (and often omitted) in files containing only PHP code. This prevents accidental whitespace issues that can cause "headers already sent" errors.

### The Echo Statement

Use `echo` to output text to the browser:

```php
<?php
echo "Hello, World!";
?>
```

This sends the string "Hello, World!" as part of the HTML response.

### Embedding PHP in HTML

PHP shines when mixed with HTML to create dynamic pages:

```php
<!DOCTYPE html>
<html>
<head>
    <title>My First PHP Page</title>
</head>
<body>
    <h1><?php echo "Welcome to PHP!"; ?></h1>
    <p>The current year is: <?php echo date("Y"); ?></p>
</body>
</html>
```

The PHP interpreter processes only the code between `<?php` and `?>` tags. Everything else is sent to the browser as plain HTML.

### Short Echo Syntax

For simple output within HTML, use the shorthand `<?=`:

```php
<p>Current time: <?= date("H:i:s") ?></p>
<!-- Equivalent to: <?php echo date("H:i:s"); ?> -->
```

This is cleaner and is the preferred style in modern PHP templates.

### Comments in PHP

PHP supports three comment styles:

```php
<?php
// Single-line comment

# Another single-line comment (shell-style)

/*
  Multi-line comment
  for longer explanations
*/

/**
 * DocBlock comment
 * Used for documentation and IDE tooling
 * @param string $name The user's name
 */
```

### Case Sensitivity

A subtle but important rule: keywords are case-insensitive, but variable names are not.

```php
<?php
ECHO "This works!";  // Keywords are case-insensitive
$name = "Alice";
// echo $Name;  // Error! Variables ARE case-sensitive
```

Always use consistent casing to avoid confusion.

## Common Pitfalls
1. **Forgetting the semicolon** — Every PHP statement must end with a semicolon (`;`). Missing it causes a parse error.
2. **Leaving the closing `?>` tag in pure PHP files** — If there is whitespace or a newline after `?>`, PHP will output it. This can break headers, cookies, and JSON responses. Omit `?>` in files that contain only PHP.
3. **Mixing up `$name` and `$Name`** — PHP variables are case-sensitive. These are two completely different variables.

## Best Practices
1. **Omit the closing `?>` tag** — In files that contain only PHP code, always leave off the closing tag to prevent accidental whitespace output.
2. **Use `<?= ?>` for output in templates** — It is shorter, cleaner, and universally supported since PHP 5.4.
3. **Use DocBlock comments for functions and classes** — They power IDE autocompletion and generate documentation automatically.

## Summary
- PHP code lives inside `<?php ... ?>` tags; the closing tag is optional in pure PHP files.
- `echo` outputs text and HTML to the browser.
- Use `<?= ?>` as shorthand for simple echo statements in HTML.
- PHP keywords are case-insensitive, but variable names are case-sensitive.
- Always end statements with a semicolon.

## Code Examples

**A simple PHP script that outputs text and the current date — notice the dot operator for string concatenation**

```php
<?php
// hello.php - Your first PHP script
// This outputs text and the current date to the browser

echo "Hello, World!";
echo "<br>";
echo "The current date is: " . date("Y-m-d");

// The dot (.) operator concatenates strings in PHP
// date() is a built-in function that formats the current date
?>
```


## Resources

- [PHP Basic Syntax](https://www.php.net/manual/en/language.basic-syntax.php) — Official guide to PHP's basic syntax rules

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*