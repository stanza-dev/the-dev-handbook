---
source_course: "php-essentials"
source_lesson: "php-essentials-first-php-script"
---

# Writing Your First PHP Script

Let's write your first PHP program! PHP code is embedded within special tags that tell the server to process the code.

## PHP Tags

PHP code must be enclosed in opening and closing tags:

```php
<?php
// Your PHP code goes here
?>
```

**Important**: The closing `?>` tag is optional (and often omitted) in files containing only PHP code. This prevents accidental whitespace issues.

## The Echo Statement

The `echo` statement outputs text to the browser:

```php
<?php
echo "Hello, World!";
?>
```

## Embedding PHP in HTML

PHP shines when mixed with HTML:

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

## Short Echo Syntax

For simple output within HTML, use the shorthand `<?=`:

```php
<p>Current time: <?= date("H:i:s") ?></p>
<!-- Equivalent to: <?php echo date("H:i:s"); ?> -->
```

## Comments in PHP

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
 * Used for documentation
 * @param string $name The user's name
 */
```

## Case Sensitivity

- **Keywords** (`if`, `else`, `echo`, `class`): Case-**insensitive**
- **Variable names**: Case-**sensitive** (`$name` ≠ `$Name`)

```php
<?php
ECHO "This works!";  // Keywords are case-insensitive
$name = "Alice";
echo $Name;  // Error! Variables ARE case-sensitive
```

## Code Examples

**A simple PHP script that outputs text and the current date**

```php
<?php
// hello.php - Your first PHP script
echo "Hello, World!";
echo "<br>";
echo "The current date is: " . date("Y-m-d");
?>
```


## Resources

- [PHP Basic Syntax](https://www.php.net/manual/en/language.basic-syntax.php) — Official guide to PHP's basic syntax rules

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*