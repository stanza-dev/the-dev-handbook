---
source_course: "php-testing"
source_lesson: "php-testing-code-standards"
---

# Code Standards with PHP_CodeSniffer

PHP_CodeSniffer enforces consistent coding standards across your codebase.

## Installation

```bash
composer require --dev squizlabs/php_codesniffer
```

## Basic Usage

```bash
# Check code
./vendor/bin/phpcs src/

# Fix automatically
./vendor/bin/phpcbf src/

# Specific standard
./vendor/bin/phpcs --standard=PSR12 src/
```

## Configuration (phpcs.xml)

```xml
<?xml version="1.0"?>
<ruleset name="MyProject">
    <description>Coding standard for MyProject</description>
    
    <file>src</file>
    <file>tests</file>
    
    <exclude-pattern>*/vendor/*</exclude-pattern>
    <exclude-pattern>*/cache/*</exclude-pattern>
    
    <!-- Use PSR-12 as base -->
    <rule ref="PSR12"/>
    
    <!-- Custom rules -->
    <rule ref="Generic.Files.LineLength">
        <properties>
            <property name="lineLimit" value="120"/>
            <property name="absoluteLineLimit" value="150"/>
        </properties>
    </rule>
    
    <!-- Exclude specific rules -->
    <rule ref="PSR12.Files.FileHeader.SpacingAfterBlock">
        <exclude-pattern>*/tests/*</exclude-pattern>
    </rule>
</ruleset>
```

## Common Standards

- **PSR-1**: Basic coding standard
- **PSR-12**: Extended coding style (recommended)
- **Squiz**: Comprehensive standard
- **PEAR**: Legacy standard

## Example Violations

```php
<?php
// Violation: Opening brace on wrong line (PSR-12)
class User
{ // Should be on same line
    // Violation: Missing visibility
    function getName() { return $this->name; }
    
    // Violation: Inconsistent spacing
    public function setName( string $name ):void {
        $this->name=$name;  // Missing spaces around =
    }
}

// Fixed:
class User {
    private string $name;
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function setName(string $name): void
    {
        $this->name = $name;
    }
}
```

## Combining Tools

```json
// composer.json scripts
{
    "scripts": {
        "test": "phpunit",
        "analyse": "phpstan analyse",
        "cs-check": "phpcs",
        "cs-fix": "phpcbf",
        "quality": [
            "@cs-check",
            "@analyse",
            "@test"
        ]
    }
}
```

```bash
composer quality  # Run all checks
```

## Code Examples

**Quality gate script combining all analysis tools**

```php
<?php
declare(strict_types=1);

// CI pipeline configuration (GitHub Actions example)
// .github/workflows/ci.yml
/*
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          coverage: xdebug
      
      - name: Install dependencies
        run: composer install --prefer-dist --no-progress
      
      - name: Check code style
        run: vendor/bin/phpcs
      
      - name: Static analysis
        run: vendor/bin/phpstan analyse --no-progress
      
      - name: Run tests
        run: vendor/bin/phpunit --coverage-text
*/

// Example quality gate script
class QualityGate
{
    private array $results = [];
    
    public function run(): int
    {
        $this->check('PHP Syntax', $this->checkSyntax());
        $this->check('Code Style (PHPCS)', $this->runPhpcs());
        $this->check('Static Analysis (PHPStan)', $this->runPhpstan());
        $this->check('Unit Tests (PHPUnit)', $this->runTests());
        
        $this->printResults();
        
        return $this->hasFailures() ? 1 : 0;
    }
    
    private function checkSyntax(): bool
    {
        exec('find src tests -name "*.php" -exec php -l {} \; 2>&1', $output, $code);
        return $code === 0;
    }
    
    private function runPhpcs(): bool
    {
        exec('./vendor/bin/phpcs --report=summary 2>&1', $output, $code);
        return $code === 0;
    }
    
    private function runPhpstan(): bool
    {
        exec('./vendor/bin/phpstan analyse --no-progress 2>&1', $output, $code);
        return $code === 0;
    }
    
    private function runTests(): bool
    {
        exec('./vendor/bin/phpunit --testdox 2>&1', $output, $code);
        return $code === 0;
    }
    
    private function check(string $name, bool $passed): void
    {
        $this->results[$name] = $passed;
    }
    
    private function printResults(): void
    {
        echo "\n=== Quality Gate Results ===\n";
        foreach ($this->results as $name => $passed) {
            $status = $passed ? '✅ PASS' : '❌ FAIL';
            echo "$status: $name\n";
        }
    }
    
    private function hasFailures(): bool
    {
        return in_array(false, $this->results, true);
    }
}

// Run: php quality-gate.php
$gate = new QualityGate();
exit($gate->run());
?>
```


## Resources

- [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) — PHP_CodeSniffer documentation

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*