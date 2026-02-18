---
source_course: "php-security"
source_lesson: "php-security-dependency-scanning"
---

# Dependency Security Scanning

Your application is only as secure as its dependencies. Regularly scan for known vulnerabilities in third-party packages.

## Composer Audit

```bash
# Built-in security audit (Composer 2.4+)
composer audit

# Output shows:
# - Package name
# - Installed version
# - CVE identifiers
# - Severity level
# - Advisory details

# Exit code: non-zero if vulnerabilities found (useful for CI)
composer audit && echo 'No vulnerabilities'
```

## Automated CI Integration

```yaml
# GitHub Actions
name: Security
on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.4'
          
      - name: Install dependencies
        run: composer install --no-dev
        
      - name: Security audit
        run: composer audit --format=json > audit.json
        
      - name: Upload report
        uses: actions/upload-artifact@v3
        with:
          name: security-audit
          path: audit.json
```

## PHP Security Advisories Database

```bash
# Install local security checker
composer require --dev roave/security-advisories:dev-latest

# This package has no code - it conflicts with vulnerable packages
# Composer will refuse to install known vulnerable versions
```

## Static Analysis for Security

```bash
# PHPStan with security rules
composer require --dev phpstan/phpstan
composer require --dev phpstan/phpstan-strict-rules

# psalm with taint analysis
composer require --dev vimeo/psalm

# Run taint analysis (tracks user input flow)
./vendor/bin/psalm --taint-analysis
```

## Psalm Taint Analysis

```php
<?php
/**
 * @psalm-taint-source input
 */
function getUserInput(): string
{
    return $_GET['data'];
}

/**
 * @psalm-taint-sink sql
 */
function executeQuery(string $sql): void
{
    // If tainted data reaches here, Psalm reports it
    $pdo->query($sql);
}

// Psalm will flag this as tainted data flowing to SQL sink
$input = getUserInput();
executeQuery("SELECT * FROM users WHERE name = '$input'");
```

## Security Testing Checklist

```php
<?php
class SecurityTestSuite
{
    public function run(): array
    {
        return [
            'sql_injection' => $this->testSqlInjection(),
            'xss' => $this->testXss(),
            'csrf' => $this->testCsrf(),
            'auth_bypass' => $this->testAuthBypass(),
            'path_traversal' => $this->testPathTraversal(),
            'open_redirect' => $this->testOpenRedirect(),
        ];
    }
    
    private function testSqlInjection(): bool
    {
        $payloads = ["' OR '1'='1", "1; DROP TABLE users", "1 UNION SELECT"];
        
        foreach ($payloads as $payload) {
            $response = $this->request('/api/users/' . urlencode($payload));
            
            // Check for SQL error messages in response
            if (preg_match('/SQL|syntax|mysql|ORA-/i', $response)) {
                return false;  // Vulnerability found
            }
        }
        
        return true;
    }
    
    // Additional test methods...
}
```

## Resources

- [Composer Audit](https://getcomposer.org/doc/03-cli.md#audit) — Composer security audit command

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*