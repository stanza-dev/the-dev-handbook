---
source_course: "php-essentials"
source_lesson: "php-essentials-conditional-statements"
---

# Conditional Statements

Conditional statements allow your code to make decisions and execute different blocks based on conditions.

## The `if` Statement

The simplest form of conditional:

```php
<?php
$age = 18;

if ($age >= 18) {
    echo "You are an adult.";
}
```

## `if-else` Statement

Handle two possible outcomes:

```php
<?php
$score = 75;

if ($score >= 60) {
    echo "You passed!";
} else {
    echo "You failed.";
}
```

## `if-elseif-else` Chain

Handle multiple conditions:

```php
<?php
$grade = 85;

if ($grade >= 90) {
    echo "A - Excellent!";
} elseif ($grade >= 80) {
    echo "B - Good job!";
} elseif ($grade >= 70) {
    echo "C - Satisfactory";
} elseif ($grade >= 60) {
    echo "D - Needs improvement";
} else {
    echo "F - Failed";
}
```

## Ternary Operator

A shorthand for simple if-else:

```php
<?php
$age = 20;

// Ternary: condition ? true_value : false_value
$status = $age >= 18 ? "adult" : "minor";

// Equivalent to:
if ($age >= 18) {
    $status = "adult";
} else {
    $status = "minor";
}
```

## Null Coalescing Operator (`??`)

Return first non-null value:

```php
<?php
$username = $_GET['user'] ?? 'Guest';
// If $_GET['user'] is null/undefined, use 'Guest'

// Can chain multiple:
$name = $firstName ?? $nickname ?? 'Anonymous';
```

## Null Coalescing Assignment (`??=`)

Assign only if null (PHP 7.4+):

```php
<?php
$config['timeout'] ??= 30;
// Only sets to 30 if $config['timeout'] is null
```

## Comparison Operators

| Operator | Description | Example |
|----------|-------------|--------|
| `==` | Equal value | `5 == "5"` → true |
| `===` | Identical (value & type) | `5 === "5"` → false |
| `!=` | Not equal | `5 != 3` → true |
| `!==` | Not identical | `5 !== "5"` → true |
| `<>` | Not equal (alt) | `5 <> 3` → true |
| `<` | Less than | `3 < 5` → true |
| `>` | Greater than | `5 > 3` → true |
| `<=` | Less or equal | `3 <= 3` → true |
| `>=` | Greater or equal | `5 >= 5` → true |
| `<=>` | Spaceship | `1 <=> 2` → -1 |

## Code Examples

**Practical conditional logic for user authentication**

```php
<?php
// Real-world example: User authentication check
$user = $_SESSION['user'] ?? null;
$role = $user['role'] ?? 'guest';

if ($role === 'admin') {
    echo "Welcome, Administrator!";
    // Show admin dashboard
} elseif ($role === 'member') {
    echo "Welcome back, Member!";
    // Show member area
} else {
    echo "Please log in to continue.";
    // Show login form
}

// Using ternary for simple display logic
$greeting = $user ? "Hello, {$user['name']}" : "Hello, Guest";
echo $greeting;
?>
```


## Resources

- [PHP Control Structures](https://www.php.net/manual/en/language.control-structures.php) — Official guide to all PHP control structures

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*