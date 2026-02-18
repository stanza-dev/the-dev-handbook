---
source_course: "php-essentials"
source_lesson: "php-essentials-variable-scope"
---

# Variable Scope in PHP

Scope determines where a variable can be accessed. Understanding scope prevents bugs and helps organize code.

## Local Scope

Variables inside functions are local:

```php
<?php
function test() {
    $localVar = "I'm local!";
    echo $localVar;  // Works
}

test();
echo $localVar;  // Error! Not defined outside
```

## Global Scope

Variables outside functions have global scope:

```php
<?php
$globalVar = "I'm global!";

function test() {
    echo $globalVar;  // Error! Not accessible
}
```

## Accessing Globals (Use Sparingly)

### Using `global` keyword:

```php
<?php
$counter = 0;

function increment() {
    global $counter;  // Reference the global
    $counter++;
}

increment();
echo $counter;  // 1
```

### Using `$GLOBALS` array:

```php
<?php
$name = "John";

function greet() {
    echo "Hello, " . $GLOBALS['name'];
}

greet();  // Hello, John
```

> **Warning**: Global variables make code harder to test and maintain. Prefer passing parameters instead!

## Static Variables

Retain value between function calls:

```php
<?php
function countCalls() {
    static $count = 0;  // Initialized once
    $count++;
    return $count;
}

echo countCalls();  // 1
echo countCalls();  // 2
echo countCalls();  // 3
```

## Best Practice: Pass Parameters

Instead of globals, pass what you need:

```php
<?php
// Bad: Using global
$config = ['debug' => true];

function log($message) {
    global $config;
    if ($config['debug']) {
        echo $message;
    }
}

// Good: Pass as parameter
function log(string $message, bool $debug = false): void {
    if ($debug) {
        echo $message;
    }
}
```

## Resources

- [Variable Scope](https://www.php.net/manual/en/language.variables.scope.php) — Official documentation on PHP variable scope

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*