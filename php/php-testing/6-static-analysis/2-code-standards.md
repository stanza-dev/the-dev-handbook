---
source_course: "php-testing"
source_lesson: "php-testing-code-standards"
---

# PHP CS Fixer and Coding Standards

## Introduction
Consistent code style makes projects easier to read, review, and maintain. PHP CS Fixer automatically formats your PHP code to follow coding standards like PSR-12. Instead of debating brace placement in code reviews, you configure the tool once and let it enforce the rules for you.

## Key Concepts
- **PHP CS Fixer**: A tool that automatically fixes PHP code style to conform to a set of rules. Created by Fabien Potencier (Symfony creator) and Dariusz Rumiński.
- **PSR-12**: The PHP-FIG Extended Coding Style standard. It builds on PSR-1 and defines rules for formatting classes, methods, control structures, and more.
- **Coding Standard**: A set of conventions for how code should be formatted — indentation, brace placement, spacing, and naming.
- **Fixer Rule**: An individual formatting rule in PHP CS Fixer (e.g., `braces_position`, `single_quote`).

## Real World Context
Every major PHP framework (Symfony, Laravel, Drupal) enforces a coding standard. On a team, inconsistent formatting creates noisy diffs and wastes time in code reviews arguing about style. PHP CS Fixer eliminates this entirely. You run it before committing, and every file follows the same conventions automatically.

## Deep Dive

### Installation
Install PHP CS Fixer as a dev dependency.

```bash
composer require --dev friendsofphp/php-cs-fixer
```

Run it on your source directory to see what it would fix.

```bash
# Dry run: show what would change without modifying files
./vendor/bin/php-cs-fixer fix src --dry-run --diff

# Actually fix the files
./vendor/bin/php-cs-fixer fix src
```

The `--dry-run` flag shows proposed changes without applying them, which is useful in CI to fail the build on style violations.

### PSR-12 Brace Rules (Critical)
PSR-12 has specific rules about where opening braces go. Getting these right is essential.

**Classes, interfaces, and traits: opening brace on a NEW line.**

```php
<?php
// CORRECT: class brace on its own line
class UserRepository implements RepositoryInterface
{
    // ...
}

// WRONG: class brace on the same line
class UserRepository implements RepositoryInterface {
    // ...
}
```

**Methods: opening brace on a NEW line.**

```php
<?php
class OrderService
{
    // CORRECT: method brace on its own line
    public function calculateTotal(array $items): float
    {
        return array_sum($items);
    }

    // WRONG: method brace on the same line
    public function calculateTotal(array $items): float {
        return array_sum($items);
    }
}
```

**Control structures (if, for, while, switch): opening brace on the SAME line.**

```php
<?php
// CORRECT: control structure brace on the SAME line
if ($order->isValid()) {
    $order->process();
} elseif ($order->isPending()) {
    $order->retry();
} else {
    throw new InvalidOrderException();
}

for ($i = 0; $i < count($items); $i++) {
    $this->processItem($items[$i]);
}

while ($queue->hasMessages()) {
    $message = $queue->dequeue();
    $this->handle($message);
}

// WRONG: control structure brace on a new line
if ($order->isValid())
{
    $order->process();
}
```

This distinction — new line for classes and methods, same line for control structures — is the most commonly confused PSR-12 rule.

### Configuration File
Create a `.php-cs-fixer.dist.php` file in your project root to define your ruleset.

```php
<?php
use PhpCsFixer\Config;
use PhpCsFixer\Finder;

$finder = Finder::create()
    ->in(__DIR__ . '/src')
    ->in(__DIR__ . '/tests')
    ->name('*.php');

return (new Config())
    ->setRules([
        '@PSR12' => true,
        'array_syntax' => ['syntax' => 'short'],
        'single_quote' => true,
        'no_unused_imports' => true,
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
        'trailing_comma_in_multiline' => true,
    ])
    ->setFinder($finder)
    ->setRiskyAllowed(false);
```

The `@PSR12` preset applies all PSR-12 rules. The additional rules enforce short array syntax, single quotes, clean imports, and trailing commas in multi-line structures.

### Combining with Git Hooks
You can run PHP CS Fixer automatically before every commit using a Git pre-commit hook.

```bash
#!/bin/sh
# .git/hooks/pre-commit
./vendor/bin/php-cs-fixer fix --dry-run --diff
if [ $? -ne 0 ]; then
    echo "Code style violations found. Run: ./vendor/bin/php-cs-fixer fix"
    exit 1
fi
```

This prevents poorly formatted code from entering the repository. Developers who forget to run the fixer will be reminded automatically.

## Common Pitfalls
1. **Confusing class braces with control structure braces** — PSR-12 puts class and method braces on a new line, but control structure braces on the same line. Mixing these up is the most common PSR-12 violation. Let PHP CS Fixer handle it automatically.
2. **Not committing the configuration file** — If `.php-cs-fixer.dist.php` is not in version control, each developer may use different rules. Always commit the configuration so the entire team shares the same standard.

## Best Practices
1. **Use the @PSR12 preset as your foundation** — Start with PSR-12 and add project-specific rules on top. This ensures compatibility with the broader PHP ecosystem.
2. **Run the fixer in CI as a dry-run** — Add `php-cs-fixer fix --dry-run --diff` to your CI pipeline. This fails the build when code style violations exist, enforcing consistency without relying on developers to remember.

## Summary
- PHP CS Fixer automatically formats PHP code to follow coding standards like PSR-12.
- PSR-12 requires class and method braces on a new line, but control structure braces on the same line.
- Configure rules in `.php-cs-fixer.dist.php` and commit it to version control.
- Use `--dry-run` in CI to enforce style and `fix` locally to auto-correct violations.
- Combining PHP CS Fixer with Git hooks prevents style violations from being committed.

## Code Examples

**PSR-12 formatted class showing correct brace placement: new line for class/method declarations, same line for control structures**

```php
<?php
declare(strict_types=1);

namespace App\Service;

use App\Entity\Order;
use App\Repository\OrderRepository;

// PSR-12 compliant class: opening brace on a NEW line
class OrderProcessor
{
    public function __construct(
        private readonly OrderRepository $orderRepository,
    ) {
    }

    // Method opening brace on a NEW line
    public function processOrder(Order $order): bool
    {
        // Control structure braces on the SAME line
        if (!$order->isValid()) {
            return false;
        }

        foreach ($order->getItems() as $item) {
            if ($item->getQuantity() <= 0) {
                continue;
            }

            $this->fulfillItem($item);
        }

        return true;
    }

    // Another method: brace on a NEW line
    private function fulfillItem(OrderItem $item): void
    {
        // while loop: brace on the SAME line
        while ($item->hasPendingStock()) {
            $item->allocateNextUnit();
        }
    }
}
```


## Resources

- [PHP CS Fixer Documentation](https://cs.symfony.com/) — Official PHP CS Fixer website with rule documentation and configuration guide
- [PSR-12: Extended Coding Style](https://www.php-fig.org/psr/psr-12/) — The official PSR-12 specification from PHP-FIG

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*