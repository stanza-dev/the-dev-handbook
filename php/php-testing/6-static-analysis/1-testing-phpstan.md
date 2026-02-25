---
source_course: "php-testing"
source_lesson: "php-testing-phpstan"
---

# Introduction to PHPStan

## Introduction
Static analysis finds bugs in your code without running it. PHPStan reads your PHP files and checks for type errors, undefined variables, incorrect method calls, and more. Think of it as a spell-checker for code: it catches mistakes before they reach production.

## Key Concepts
- **Static Analysis**: Examining source code without executing it, to detect errors, type mismatches, and unreachable code paths.
- **PHPStan**: The most widely used static analysis tool for PHP, created by Ondrej Mirtes. It checks your code against rules of increasing strictness.
- **Rule Levels (0–9)**: PHPStan's ten levels of strictness. Level 0 catches only the most basic errors; level 9 enforces the strictest type safety.
- **Baseline**: A snapshot of existing errors that PHPStan ignores, allowing you to adopt strict levels on legacy codebases without fixing every old issue first.

## Real World Context
PHPStan is used by Symfony, Laravel, Doctrine, and thousands of PHP projects. Running it in CI catches bugs that unit tests miss: calling methods that do not exist, passing wrong argument types, or using variables that might be null. A 2023 survey showed that over 60% of PHP projects use some form of static analysis, with PHPStan being the most popular choice.

## Deep Dive

### Installation
Install PHPStan as a dev dependency via Composer.

```bash
composer require --dev phpstan/phpstan
```

After installation, you can run PHPStan from the command line.

```bash
./vendor/bin/phpstan analyse src
```

This analyzes every PHP file in the `src` directory and reports any issues found.

### Understanding Rule Levels
PHPStan has ten levels, numbered 0 through 9. Each level includes all checks from lower levels plus additional, stricter rules.

```bash
# Run at level 0 (most lenient)
./vendor/bin/phpstan analyse src --level=0

# Run at level 5 (moderate strictness)
./vendor/bin/phpstan analyse src --level=5

# Run at level 9 (maximum strictness)
./vendor/bin/phpstan analyse src --level=9
```

Here is what each level range focuses on:

- **Level 0–1**: Basic checks — unknown classes, unknown functions, wrong number of arguments.
- **Level 2–3**: Unknown methods on known classes, return type validation.
- **Level 4–5**: Dead code detection, type checking in conditions, strict comparison checks.
- **Level 6–7**: Missing typehints reported, strict return types, union type handling.
- **Level 8–9**: Nullable types, mixed type restrictions, strictest possible analysis.

### Configuration File
For real projects, use a configuration file instead of command-line flags. PHPStan looks for `phpstan.neon` or `phpstan.neon.dist` by default.

```yaml
# phpstan.neon
parameters:
    level: 6
    paths:
        - src
        - tests
    excludePaths:
        - src/Legacy
    checkMissingIterableValueType: false
```

The NEON format is similar to YAML. The `parameters` section configures PHPStan's behavior, `paths` lists directories to analyze, and `excludePaths` skips directories you are not ready to fix yet.

### Example: Catching Bugs
Consider this code that looks correct but has a subtle bug.

```php
<?php
class OrderService
{
    public function calculateTotal(array $items): float
    {
        $total = 0;
        foreach ($items as $item) {
            $total += $item->getPrice();  // What if $item is not an object?
        }
        return $total;
    }

    public function findOrder(int $id): Order
    {
        $order = $this->repository->find($id);
        return $order;  // Could be null if not found!
    }
}
```

PHPStan at level 6 would flag both issues. It reports that `$item` has no type information (could be anything), and that `findOrder` promises to return `Order` but might return `null`.

The fixed version adds proper type declarations.

```php
<?php
class OrderService
{
    /**
     * @param array<OrderItem> $items
     */
    public function calculateTotal(array $items): float
    {
        $total = 0.0;
        foreach ($items as $item) {
            $total += $item->getPrice();  // PHPStan now knows $item is OrderItem
        }
        return $total;
    }

    public function findOrder(int $id): ?Order
    {
        return $this->repository->find($id);  // Return type is now nullable
    }
}
```

With proper PHPDoc annotations and nullable return types, PHPStan verifies that every code path is type-safe.

### Using a Baseline
When adopting PHPStan on a legacy project, generating a baseline lets you enforce strict rules on new code without fixing hundreds of existing issues first.

```bash
./vendor/bin/phpstan analyse src --level=8 --generate-baseline
```

This creates a `phpstan-baseline.neon` file listing all current errors. Include it in your configuration so PHPStan ignores those specific errors while catching any new ones.

```yaml
# phpstan.neon
includes:
    - phpstan-baseline.neon
parameters:
    level: 8
    paths:
        - src
```

Over time, you can fix baseline errors and remove them from the file, gradually cleaning up the codebase.

## Common Pitfalls
1. **Starting at level 9 on a legacy project** — You will be overwhelmed with hundreds of errors. Start at level 0 or 1, fix those issues, and increase the level gradually. Use a baseline for existing errors.
2. **Ignoring PHPStan errors with comments** — Adding `@phpstan-ignore-next-line` everywhere defeats the purpose. Use ignores sparingly and only for genuine false positives, never to hide real bugs.

## Best Practices
1. **Run PHPStan in CI** — Make PHPStan a required check in your CI pipeline. This prevents anyone from merging code that introduces new static analysis errors.
2. **Increase levels incrementally** — Adopt PHPStan at level 0, stabilize, then bump to level 1, and so on. Each level increase is a small, manageable step toward stricter type safety.

## Summary
- PHPStan analyzes PHP code without executing it, catching type errors and bugs at development time.
- Rule levels 0 through 9 provide increasing strictness, from basic class existence checks to full type safety.
- Configure PHPStan with a `phpstan.neon` file for reproducible analysis across your team.
- Use baselines to adopt strict analysis on legacy codebases without fixing every existing error upfront.
- Running PHPStan in CI prevents new type errors from reaching production.

## Code Examples

**Before and after: fixing code to pass PHPStan level 6 by adding proper type declarations and PHPDoc annotations**

```php
<?php
declare(strict_types=1);

// BEFORE: Code that PHPStan would flag at level 6
class UserService
{
    // No return type declared
    public function findUser(int $id)
    {
        $user = $this->repository->find($id);
        return $user; // Could return null
    }

    // No parameter type info for array items
    public function getEmails(array $users): array
    {
        $emails = [];
        foreach ($users as $user) {
            $emails[] = $user->getEmail(); // What type is $user?
        }
        return $emails;
    }
}

// AFTER: PHPStan-clean code with proper types
class UserService
{
    public function findUser(int $id): ?User
    {
        return $this->repository->find($id); // Nullable return is explicit
    }

    /**
     * @param array<User> $users
     * @return array<string>
     */
    public function getEmails(array $users): array
    {
        $emails = [];
        foreach ($users as $user) {
            $emails[] = $user->getEmail(); // PHPStan knows $user is User
        }
        return $emails;
    }
}
```


## Resources

- [PHPStan Documentation](https://phpstan.org/user-guide/getting-started) — Official PHPStan getting started guide with installation and configuration
- [PHP Type Declarations](https://www.php.net/manual/en/language.types.declarations.php) — PHP reference for type declarations that enable effective static analysis

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*