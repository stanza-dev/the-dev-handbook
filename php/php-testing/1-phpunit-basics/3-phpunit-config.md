---
source_course: "php-testing"
source_lesson: "php-testing-phpunit-config"
---

# Running and Configuring PHPUnit

## Introduction
Installing PHPUnit is only the first step. To get the most out of your test suite you need a well-tuned `phpunit.xml` configuration file and a solid understanding of the command-line runner. This lesson covers the configuration options that matter in day-to-day development, how to run targeted subsets of tests, and how to customise output so failures are easy to diagnose.

## Key Concepts
- **phpunit.xml**: The central configuration file that defines test suites, bootstrap scripts, colour output, and caching behaviour for PHPUnit.
- **Test suite**: A named collection of test directories defined inside `<testsuites>`. Suites let you run unit tests separately from integration tests.
- **Bootstrap**: A PHP file (usually Composer's `vendor/autoload.php`) that PHPUnit loads before any test runs, ensuring your classes are autoloaded.
- **--filter**: A command-line flag that runs only the tests whose name matches a given pattern.
- **--stop-on-failure**: A flag that halts the entire run on the first failing test, saving time during active debugging.

## Real World Context
On a large codebase with thousands of tests, running the full suite after every code change is impractical. A developer working on the `InvoiceService` can use `--filter InvoiceService` to execute only the relevant tests in under a second. Combined with `--stop-on-failure`, this creates a tight feedback loop that catches regressions instantly without waiting for unrelated tests to finish.

## Deep Dive

### Anatomy of phpunit.xml

A production-ready configuration covers suites, source directories for coverage, and display settings. Here is a comprehensive example for PHPUnit 12:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
         cacheDirectory=".phpunit.cache"
         executionOrder="random"
         failOnWarning="true"
         failOnRisky="true"
         beStrictAboutTestsThatDoNotTestAnything="true">
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

The `colors="true"` attribute enables coloured terminal output so passes appear green and failures appear red. The `executionOrder="random"` attribute shuffles test order on each run, which helps uncover hidden dependencies between tests. The strictness flags ensure that tests which produce warnings or do not contain assertions are reported as failures.

### Defining Multiple Test Suites

Suites group tests by purpose. You can define as many as you need:

```xml
<testsuites>
    <testsuite name="Unit">
        <directory>tests/Unit</directory>
    </testsuite>
    <testsuite name="Integration">
        <directory>tests/Integration</directory>
    </testsuite>
    <testsuite name="Feature">
        <directory>tests/Feature</directory>
    </testsuite>
</testsuites>
```

Run a specific suite with the `--testsuite` flag:

```bash
# Run only unit tests
./vendor/bin/phpunit --testsuite Unit

# Run only integration tests
./vendor/bin/phpunit --testsuite Integration
```

This separation is essential in CI pipelines where unit tests run on every commit and slower integration tests run on merge requests.

### Running Subsets with --filter

The `--filter` flag accepts a string or regex matched against the fully qualified test name. This is the most common flag during development:

```bash
# Run all tests in CalculatorTest
./vendor/bin/phpunit --filter CalculatorTest

# Run a single test method
./vendor/bin/phpunit --filter 'CalculatorTest::addsTwoPositiveNumbers'

# Run tests matching a pattern
./vendor/bin/phpunit --filter 'adds|subtracts'
```

The filter matches against the class name, method name, and data-provider dataset names, giving you fine-grained control over what executes.

### Useful Command-Line Flags

PHPUnit provides several flags that improve the development experience:

```bash
# Stop on first failure — ideal during active debugging
./vendor/bin/phpunit --stop-on-failure

# Display detailed output for every test (not just dots)
./vendor/bin/phpunit --display-deprecations --display-warnings

# Run tests in random order to catch hidden dependencies
./vendor/bin/phpunit --order-by random

# Combine flags for a focused debugging session
./vendor/bin/phpunit --filter InvoiceService --stop-on-failure
```

The `--stop-on-failure` flag is especially valuable when you know something is broken and want to fix the first failure before seeing the cascading effects on other tests.

### The Bootstrap File

The `bootstrap` attribute specifies a PHP file loaded before any test executes. In most projects this is Composer's autoloader:

```xml
<phpunit bootstrap="vendor/autoload.php">
```

You can point to a custom bootstrap file if you need to set environment variables, configure a test database, or register custom autoloaders:

```php
<?php
// tests/bootstrap.php

require __DIR__ . '/../vendor/autoload.php';

// Set timezone for consistent date tests
date_default_timezone_set('UTC');

// Load test-specific environment variables
$_ENV['APP_ENV'] = 'testing';
$_ENV['DB_DATABASE'] = 'app_test';
```

Then reference it in your configuration:

```xml
<phpunit bootstrap="tests/bootstrap.php">
```

This ensures every test runs in a consistent, predictable environment.

## Common Pitfalls
1. **Not setting `colors="true"`** — Without colour output, it is hard to spot failures in a wall of text. Always enable colours in `phpunit.xml` so green means pass and red means failure.
2. **Relying on test execution order** — Tests that depend on running in a specific order will break under `executionOrder="random"`. Each test must be fully independent. If a test fails only when order is randomised, it has a hidden dependency on another test's side effects.

## Best Practices
1. **Commit `phpunit.xml` to version control** — Every developer and CI environment should use the same configuration. Add `.phpunit.cache/` to `.gitignore` since the cache is machine-specific.
2. **Use `--stop-on-failure` during development, remove it in CI** — During active work, stopping at the first failure saves time. In CI, you want to see all failures at once so the developer can fix them in a single pass.

## Summary
- Configure PHPUnit via `phpunit.xml` with test suites, a bootstrap file, colour output, and strictness flags.
- Use `--testsuite` to run a specific group of tests and `--filter` to target individual classes or methods.
- The `--stop-on-failure` flag creates a tight feedback loop during active debugging.
- Set `executionOrder="random"` to catch hidden dependencies between tests.
- Always commit `phpunit.xml` and add `.phpunit.cache/` to `.gitignore`.

## Code Examples

**A custom bootstrap file that sets up timezone, environment variables, and error handling before PHPUnit runs any test**

```php
<?php
// tests/bootstrap.php
// Custom bootstrap file loaded before all tests run

require __DIR__ . '/../vendor/autoload.php';

// Ensure consistent timezone across all test environments
date_default_timezone_set('UTC');

// Load test environment variables
$_ENV['APP_ENV'] = 'testing';
$_ENV['DB_DATABASE'] = 'myapp_test';
$_ENV['CACHE_DRIVER'] = 'array';

// Example: register a custom error handler for tests
set_error_handler(function (int $errno, string $errstr) {
    throw new \ErrorException($errstr, 0, $errno);
});
```


## Resources

- [PHPUnit 12 – The XML Configuration File](https://docs.phpunit.de/en/12.0/configuration.html) — Complete reference for every phpunit.xml configuration option in PHPUnit 12
- [PHPUnit 12 – The Command-Line Test Runner](https://docs.phpunit.de/en/12.0/textui.html) — Official documentation for PHPUnit's command-line flags including --filter and --stop-on-failure

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*