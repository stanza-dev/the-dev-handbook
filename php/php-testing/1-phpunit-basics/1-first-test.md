---
source_course: "php-testing"
source_lesson: "php-testing-first-test"
---

# Writing Your First Test

PHPUnit is PHP's standard testing framework. Tests verify your code works as expected.

## Installing PHPUnit

```bash
composer require --dev phpunit/phpunit
```

## Test Structure

```php
<?php
// tests/CalculatorTest.php
namespace Tests;

use PHPUnit\Framework\TestCase;
use App\Calculator;

class CalculatorTest extends TestCase
{
    public function testAddTwoNumbers(): void
    {
        // Arrange: Set up the test
        $calculator = new Calculator();
        
        // Act: Perform the action
        $result = $calculator->add(2, 3);
        
        // Assert: Verify the result
        $this->assertEquals(5, $result);
    }
}
```

## The Class Under Test

```php
<?php
// src/Calculator.php
namespace App;

class Calculator
{
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }
    
    public function subtract(int $a, int $b): int
    {
        return $a - $b;
    }
    
    public function divide(int $a, int $b): float
    {
        if ($b === 0) {
            throw new \DivisionByZeroError('Cannot divide by zero');
        }
        return $a / $b;
    }
}
```

## Running Tests

```bash
# Run all tests
./vendor/bin/phpunit

# Run specific test file
./vendor/bin/phpunit tests/CalculatorTest.php

# Run specific test method
./vendor/bin/phpunit --filter testAddTwoNumbers

# Verbose output
./vendor/bin/phpunit -v
```

## PHPUnit Configuration

```xml
<!-- phpunit.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Integration">
            <directory>tests/Integration</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>src</directory>
        </include>
    </source>
</phpunit>
```

## Test Naming Conventions

```php
<?php
// Methods starting with 'test'
public function testUserCanBeCreated(): void {}

// Or using @test annotation
/** @test */
public function user_can_be_created(): void {}

// Using #[Test] attribute (PHP 8)
#[Test]
public function userCanBeCreated(): void {}
```

## Resources

- [PHPUnit Documentation](https://docs.phpunit.de/en/11.4/) — Official PHPUnit documentation

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*