---
source_course: "php-essentials"
source_lesson: "php-essentials-variable-scope"
---

# Variable Scope in PHP

## Introduction
Scope determines where a variable can be accessed in your code. PHP's scoping rules are different from many other languages, and understanding them is essential for writing bug-free functions. This lesson covers local scope, global scope, static variables, and why passing parameters is preferred over globals.

## Key Concepts
- **Local Scope**: Variables defined inside a function are only accessible within that function.
- **Global Scope**: Variables defined outside any function are only accessible outside functions (unlike many other languages).
- **`global` Keyword**: Imports a global variable into a function's local scope.
- **Static Variable (`static`)**: A local variable that retains its value between function calls.

## Real World Context
Scope bugs are among the most common PHP mistakes. In other languages like JavaScript, functions can access outer variables freely. In PHP, functions are completely isolated from the outer scope by default. This is actually a safety feature — it prevents functions from accidentally modifying global state. Professional PHP code avoids globals entirely, relying on dependency injection and parameter passing instead.

## Deep Dive

### Local Scope

Variables declared inside a function exist only within that function:

```php
<?php
function test() {
    $localVar = "I'm local!";
    echo $localVar;  // Works
}

test();
// echo $localVar;  // Error! Not defined outside
```

Once the function finishes, `$localVar` is destroyed.

### Global Scope

Variables declared outside functions are not automatically available inside them:

```php
<?php
$globalVar = "I'm global!";

function test() {
    // echo $globalVar;  // Error! Not accessible
}
```

This is a key difference from JavaScript. PHP functions do not have access to the outer scope by default.

### Accessing Globals (Use Sparingly)

The `global` keyword imports a global variable into the function's scope:

```php
<?php
$counter = 0;

function increment() {
    global $counter;
    $counter++;
}

increment();
echo $counter;  // 1
```

Alternatively, use the `$GLOBALS` superglobal array:

```php
<?php
$name = "John";

function greet() {
    echo "Hello, " . $GLOBALS['name'];
}

greet();  // Hello, John
```

Both approaches work, but both create hidden dependencies that make code harder to test and debug.

### Static Variables

A `static` variable retains its value between function calls:

```php
<?php
function countCalls() {
    static $count = 0;  // Initialized only once
    $count++;
    return $count;
}

echo countCalls();  // 1
echo countCalls();  // 2
echo countCalls();  // 3
```

The initialization (`static $count = 0`) runs only on the first call. On subsequent calls, `$count` keeps its previous value. Static variables are useful for counters, caches, and singleton-like patterns within functions.

### Best Practice: Pass Parameters Instead of Using Globals

Instead of relying on global state, pass what you need as parameters:

```php
<?php
// Bad: Using global
$config = ['debug' => true];

function logMessage($message) {
    global $config;
    if ($config['debug']) {
        echo $message;
    }
}

// Good: Pass as parameter
function logMessage(string $message, bool $debug = false): void {
    if ($debug) {
        echo $message;
    }
}
```

The second version is easier to test, has no hidden dependencies, and makes its requirements explicit through its signature.

## Common Pitfalls
1. **Expecting global variables to be available inside functions** — Unlike JavaScript, PHP functions do not close over the outer scope. You must explicitly import globals or pass them as parameters.
2. **Overusing `global`** — The `global` keyword creates hidden dependencies between functions and the outer scope, making code difficult to test and debug.
3. **Forgetting that `static` persists across calls** — A static variable is not reset between function calls. This can cause subtle bugs if you expect a fresh value each time.

## Best Practices
1. **Never use `global` in production code** — Pass everything a function needs as parameters. This makes functions pure, testable, and self-documenting.
2. **Use static variables judiciously** — They are useful for counters and simple caches, but overuse can make function behavior unpredictable.
3. **Keep functions small and focused** — If a function needs access to many external values, it is a sign the function is doing too much. Refactor it.

## Summary
- PHP functions have their own local scope and cannot access outer variables by default.
- The `global` keyword imports global variables, but should be avoided in favor of parameters.
- Static variables retain their value between function calls.
- Always prefer passing parameters to using global state.

## Code Examples

**Demonstrating scope rules — the static variable persists across calls while local variables are destroyed after each call**

```php
<?php
// Demonstrating scope: local, global, and static variables

$appName = "MyApp";  // Global scope

function getVisitorCount(): int {
    static $visitors = 0;  // Persists between calls
    $visitors++;
    return $visitors;
}

function displayWelcome(string $appName): void {
    // $appName here is the parameter, not the global
    $message = "Welcome to $appName!";  // Local scope
    echo $message . " Visitor #" . getVisitorCount() . "\n";
}

displayWelcome($appName);  // Welcome to MyApp! Visitor #1
displayWelcome($appName);  // Welcome to MyApp! Visitor #2
displayWelcome($appName);  // Welcome to MyApp! Visitor #3

// $message is not accessible here (local to displayWelcome)
// $visitors is not accessible here (local to getVisitorCount)
?>
```


## Resources

- [Variable Scope](https://www.php.net/manual/en/language.variables.scope.php) — Official documentation on PHP variable scope

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*