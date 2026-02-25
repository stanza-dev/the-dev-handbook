---
source_course: "php-security"
source_lesson: "php-security-type-safety"
---

# Type Safety & Strict Types

## Introduction
PHP's dynamic typing can introduce subtle security vulnerabilities through type juggling. Declaring `strict_types` and using strict comparison operators eliminates an entire class of bugs that attackers exploit.

## Key Concepts
- **strict_types**: A per-file declaration (`declare(strict_types=1)`) that enforces strict type checking for function arguments and return values.
- **Type Juggling**: PHP's automatic conversion between types during comparisons and operations, which can produce unexpected results.
- **Strict Comparison**: Using `===` instead of `==` to compare both value and type, preventing type coercion attacks.

## Real World Context
Type juggling vulnerabilities have been found in popular PHP frameworks and CMS platforms. An attacker can bypass authentication by exploiting loose comparisons — for example, `0 == 'password'` evaluates to `true` in PHP because the string is coerced to `0`. This allows login without knowing the password.

## Deep Dive
PHP's loose comparison operator `==` performs type coercion, which creates exploitable behavior:

```php
<?php
// Type juggling dangers
var_dump(0 == 'any-string');     // true! String coerced to 0
var_dump('0e1234' == '0e5678'); // true! Both treated as 0 in scientific notation
var_dump('' == null);            // true!
var_dump('' == false);           // true!
```

These comparisons look obviously wrong, but they appear in real code disguised within complex logic. Here is how an attacker exploits this in authentication:

```php
<?php
// VULNERABLE: Loose comparison
$token = $_GET['token'];
if ($token == $storedToken) {
    grantAccess(); // Attacker sends token=0 to match any string!
}

// SAFE: Strict comparison
if ($token === $storedToken) {
    grantAccess(); // Types must match too
}

// BEST: Use timing-safe comparison for secrets
if (hash_equals($storedToken, $token)) {
    grantAccess();
}
```

The strict comparison `===` requires both value and type to match, eliminating type juggling.

Always declare strict types at the top of every PHP file:

```php
<?php
declare(strict_types=1);

function validateAge(int $age): bool {
    return $age >= 18 && $age <= 120;
}

validateAge('25');  // TypeError! String not accepted
validateAge(25);    // Works correctly
```

With `strict_types=1`, PHP throws a `TypeError` instead of silently coercing types. This catches bugs at the point where they occur rather than propagating corrupt data.

## Common Pitfalls
1. **Using == for security-sensitive comparisons** — Always use `===` or `hash_equals()` for tokens, passwords, and authentication checks. The loose `==` operator is the source of many authentication bypasses.
2. **Forgetting strict_types is per-file** — The `declare(strict_types=1)` directive only applies to the file where it appears. You must add it to every PHP file in your project.

## Best Practices
1. **Add declare(strict_types=1) to every PHP file** — Make it the first line after `<?php` in every file. Configure your linter to enforce this.
2. **Use hash_equals() for secret comparison** — When comparing tokens, CSRF values, or any secret, use `hash_equals()` to prevent timing attacks.

## Summary
- PHP's loose comparison `==` causes type juggling vulnerabilities that attackers exploit.
- Always use strict comparison `===` or `hash_equals()` for security-sensitive checks.
- Declare `strict_types=1` in every PHP file to catch type errors early.
- Type safety is a free security improvement that prevents an entire class of bugs.

## Code Examples

**Strict types in action — PHP throws TypeError instead of silently coercing '42' to 42, catching bugs at the source**

```php
<?php
declare(strict_types=1);

// Safe input processing with strict types
function processUserId(int $id): array {
    if ($id <= 0) {
        throw new InvalidArgumentException('ID must be positive');
    }
    // $id is guaranteed to be an integer
    return ['user_id' => $id];
}

// This throws TypeError — no silent coercion
try {
    processUserId('42');  // TypeError!
} catch (TypeError $e) {
    echo 'Invalid input type';
}

// Correct usage
$result = processUserId(42);  // Works
```


## Resources

- [Type Declarations](https://www.php.net/manual/en/language.types.declarations.php) — PHP type declarations and strict_types documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*