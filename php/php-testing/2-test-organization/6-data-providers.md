---
source_course: "php-testing"
source_lesson: "php-testing-data-providers"
---

# Data Providers

## Introduction
Data providers let you run the same test logic against multiple sets of input without duplicating test methods. Instead of writing five nearly identical tests, you define one test method and a provider that supplies the data. PHPUnit 12 uses the `#[DataProvider]` attribute to connect them.

## Key Concepts
- **Data provider**: A public static method that returns an array (or iterator) of test-case datasets.
- **#[DataProvider('methodName')]**: A PHP attribute that links a test method to its data provider.
- **Named datasets**: Each dataset can have a string key that appears in test output, making failures easy to identify.

## Real World Context
Validation logic is a perfect candidate for data providers. An email validator must accept dozens of valid formats and reject dozens of invalid ones. Writing a separate test for each format would be tedious. A data provider lets you test all of them in a single, readable test method.

## Deep Dive

### Basic Data Provider

A data provider is a public static method that returns an array of arrays. Each inner array is one set of arguments passed to the test method:

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\Calculator;

class CalculatorAddTest extends TestCase
{
    #[Test]
    #[DataProvider('additionCases')]
    public function addsNumbers(int $a, int $b, int $expected): void
    {
        $calculator = new Calculator();

        $this->assertSame($expected, $calculator->add($a, $b));
    }

    public static function additionCases(): array
    {
        return [
            'positive numbers'     => [2, 3, 5],
            'negative numbers'     => [-1, -2, -3],
            'mixed signs'          => [-5, 10, 5],
            'adding zero'          => [7, 0, 7],
        ];
    }
}
```

The `#[DataProvider('additionCases')]` attribute tells PHPUnit to call `additionCases()` and run `addsNumbers()` once per dataset. The string keys (`'positive numbers'`, etc.) appear in the test output so you can see exactly which case failed.

### Named Datasets and Failure Messages

When a dataset fails, PHPUnit displays its name:

```
FAILED: CalculatorAddTest::addsNumbers with dataset "mixed signs"
```

Always name your datasets. Without names, PHPUnit shows numeric indices like `#0`, `#2`, which tell you nothing about the failing scenario.

### Providers Must Be Static

In PHPUnit 12, data provider methods must be `public static`. This is because providers are resolved before the test object is constructed, so they cannot access `$this`:

```php
<?php

// Correct — static method
public static function validEmails(): array
{
    return [
        'simple'     => ['user@example.com'],
        'subdomain'  => ['user@mail.example.com'],
        'plus tag'   => ['user+tag@example.com'],
    ];
}
```

If you declare the provider as non-static, PHPUnit 12 will skip the test with a warning.

### Complex Datasets

Providers can return objects, arrays, or any serializable value:

```php
<?php

public static function discountScenarios(): array
{
    return [
        'no discount'       => [100.00, 0,   100.00],
        '10% off'           => [100.00, 10,  90.00],
        '50% off'           => [200.00, 50,  100.00],
        'full discount'     => [75.00,  100, 0.00],
    ];
}

#[Test]
#[DataProvider('discountScenarios')]
public function appliesPercentageDiscount(
    float $originalPrice,
    int $discountPercent,
    float $expectedPrice
): void {
    $pricing = new PricingService();

    $result = $pricing->applyDiscount($originalPrice, $discountPercent);

    $this->assertSame($expectedPrice, $result);
}
```

This approach makes it trivial to add new edge cases: just append another array to the provider.

## Common Pitfalls
1. **Using `@dataProvider` docblock annotation** — PHPUnit 12 uses the `#[DataProvider]` PHP attribute. The old docblock annotation still works but is deprecated and will be removed in future versions.
2. **Making the provider non-static** — PHPUnit 12 requires provider methods to be `public static`. A non-static provider will cause the test to be skipped.

## Best Practices
1. **Name every dataset** — Use descriptive string keys so failure output immediately tells you which scenario broke. Avoid numeric indices.
2. **Keep providers close to their test** — Place the provider method directly below the test method that uses it. This makes the relationship obvious when reading the file.

## Summary
- Data providers supply multiple input sets to a single test method via `#[DataProvider('methodName')]`.
- Provider methods must be `public static` and return an array of arrays.
- Use string keys to name each dataset for readable failure output.
- Providers eliminate test duplication for validation, calculation, and formatting logic.

## Code Examples

**A data provider testing a slugify function with named datasets — each edge case is a single array entry**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\StringFormatter;

class StringFormatterTest extends TestCase
{
    #[Test]
    #[DataProvider('slugifyCases')]
    public function convertsStringToSlug(string $input, string $expected): void
    {
        $formatter = new StringFormatter();

        $this->assertSame($expected, $formatter->slugify($input));
    }

    public static function slugifyCases(): array
    {
        return [
            'lowercase words'   => ['Hello World', 'hello-world'],
            'extra spaces'      => ['  too   many  spaces  ', 'too-many-spaces'],
            'special characters' => ['Price: $9.99!', 'price-999'],
            'already a slug'    => ['my-slug', 'my-slug'],
            'unicode accents'   => ['caf\u00e9 cr\u00e8me', 'cafe-creme'],
        ];
    }
}
```


## Resources

- [PHPUnit 12 – Data Providers](https://docs.phpunit.de/en/12.0/writing-tests-for-phpunit.html#data-providers) — Official PHPUnit 12 documentation on data providers and the #[DataProvider] attribute

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*