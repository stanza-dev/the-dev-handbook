---
source_course: "php-security"
source_lesson: "php-security-dependency-scanning"
---

# Dependency Security Scanning

## Introduction
Your application is only as secure as its dependencies. Regularly scan for known vulnerabilities in third-party packages.

## Key Concepts
- **composer audit**: Built-in Composer command (since v2.4) that checks packages against the PHP Security Advisories Database.
- **CVE (Common Vulnerabilities and Exposures)**: Standardized identifiers for known security vulnerabilities.
- **Supply Chain Attacks**: Attacks targeting application dependencies rather than the application code directly.
- **Lockfile Pinning**: `composer.lock` ensures deterministic dependency resolution, preventing unexpected version changes.

## Real World Context
The 2021 Log4Shell vulnerability (CVE-2021-44228) demonstrated how a single vulnerable dependency can compromise millions of applications. PHP's ecosystem has had similar issues — the 2016 PHPMailer RCE (CVE-2016-10033) affected millions of websites. Regular dependency auditing catches these before attackers exploit them.

## Deep Dive
### Intro

Your application is only as secure as its dependencies. Regularly scan for known vulnerabilities in third-party packages.

### Composer audit

```bash
composer audit


composer audit && echo 'No vulnerabilities'
```

### Automated ci integration

```yaml
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
          php-version: '8.5'
          
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

### Php security advisories database

```bash
composer require --dev roave/security-advisories:dev-latest

```

### Static analysis for security

```bash
composer require --dev phpstan/phpstan
composer require --dev phpstan/phpstan-strict-rules

composer require --dev vimeo/psalm

./vendor/bin/psalm --taint-analysis
```

### Psalm taint analysis

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

### Security testing checklist

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

## Common Pitfalls
1. **Not running `composer audit` in CI/CD** — Vulnerabilities in dependencies can be introduced silently through `composer update`. Automate scanning on every build.
2. **Ignoring abandoned packages** — Packages no longer maintained won't receive security patches. Monitor for abandonment and plan migrations.

## Best Practices
1. **Run `composer audit` in your CI pipeline** — Fail the build on high-severity vulnerabilities to prevent deploying known-vulnerable code.
2. **Pin dependencies with `composer.lock`** — Always commit your lockfile and review `composer.lock` diffs in code review for unexpected dependency changes.

## Summary
- Run `composer audit` regularly and in CI/CD to detect known vulnerabilities in dependencies.
- Pin dependencies with `composer.lock` and review lockfile changes during code review.
- Monitor for abandoned packages and have migration plans for critical dependencies.

## Resources

- [Composer Audit](https://getcomposer.org/doc/03-cli.md#audit) — Composer security audit command

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*