---
source_course: "php-testing"
source_lesson: "php-testing-phpstan"
---

# PHPStan: Static Analysis

PHPStan finds bugs without running code by analyzing types and control flow.

## Installation

```bash
composer require --dev phpstan/phpstan
```

## Basic Usage

```bash
# Analyze src directory
./vendor/bin/phpstan analyse src

# Specific level (0-9)
./vendor/bin/phpstan analyse src --level=5

# With config file
./vendor/bin/phpstan analyse -c phpstan.neon
```

## Configuration (phpstan.neon)

```yaml
parameters:
    level: 6
    paths:
        - src
        - tests
    excludePaths:
        - src/Legacy/*
    checkMissingIterableValueType: false
```

## What PHPStan Catches

```php
<?php
class UserService
{
    public function getUser(int $id): User
    {
        $user = $this->repository->find($id);
        return $user;  // ERROR: find() returns ?User, but method returns User
    }
    
    public function processUser(User $user): void
    {
        echo $user->nmae;  // ERROR: Property 'nmae' does not exist (typo!)
    }
    
    public function calculate(int $value): int
    {
        return $value / 2;  // ERROR: Division returns float, not int
    }
}
```

## Fixing Issues

```php
<?php
class UserService
{
    public function getUser(int $id): ?User  // Fix: Allow null
    {
        return $this->repository->find($id);
    }
    
    // Or throw exception
    public function getUserOrFail(int $id): User
    {
        $user = $this->repository->find($id);
        if ($user === null) {
            throw new UserNotFoundException();
        }
        return $user;  // PHPStan knows it's not null here
    }
    
    public function calculate(int $value): int
    {
        return (int) ($value / 2);  // Fix: Cast to int
        // Or use intdiv($value, 2)
    }
}
```

## PHPStan Levels

| Level | Checks |
|-------|--------|
| 0 | Basic checks, unknown classes |
| 1 | Possibly undefined variables |
| 2 | Unknown methods on known types |
| 3 | Return types, property types |
| 4 | Basic dead code checks |
| 5 | Argument types |
| 6 | Missing typehints |
| 7 | Union types checked |
| 8 | Null checks, method calls on nullable |
| 9 | Mixed type is forbidden |

## Resources

- [PHPStan Documentation](https://phpstan.org/user-guide/getting-started) — Official PHPStan documentation

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*