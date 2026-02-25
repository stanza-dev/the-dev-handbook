---
source_course: "php-security"
source_lesson: "php-security-security-checklist"
---

# Security Audit Checklist

## Introduction
Use this checklist to audit your PHP applications for common vulnerabilities.

## Key Concepts
- **Security Audit Checklist**: A systematic review of headers, configuration, authentication, input handling, and output encoding.
- **Security Headers**: `Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options`, `X-Frame-Options` — essential HTTP response headers.
- **Automated Scanning**: Tools like OWASP ZAP, Nikto, and `composer audit` that automate common vulnerability checks.
- **Penetration Testing**: Systematic testing of your application by simulating real attacks to find vulnerabilities before attackers do.

## Real World Context
Organizations like PCI DSS require regular security audits for applications handling payment data. A pre-deployment security checklist catches common misconfigurations that automated scanners miss, like missing security headers or overly permissive CORS policies.

## Deep Dive
### Intro

Use this checklist to audit your PHP applications for common vulnerabilities.

### Input handling

- [ ] All user input is validated
- [ ] Whitelist validation where possible
- [ ] Type declarations (strict_types)
- [ ] No raw $_GET, $_POST used in SQL
- [ ] No raw user input echoed to page

### Sql security

- [ ] All queries use prepared statements
- [ ] PDO with ATTR_EMULATE_PREPARES = false
- [ ] Table/column names validated against whitelist
- [ ] Error messages don't reveal database structure

### Output security

- [ ] All output escaped with htmlspecialchars()
- [ ] Content-Security-Policy header set
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options header set

### Authentication

- [ ] Passwords hashed with password_hash()
- [ ] Session regenerated after login
- [ ] Secure session cookie settings
- [ ] Account lockout after failed attempts
- [ ] CSRF tokens on all forms

### Session management

- [ ] HTTPOnly cookies
- [ ] Secure flag on cookies
- [ ] SameSite cookie attribute
- [ ] Session timeout implemented
- [ ] Session data validated

### File handling

- [ ] File uploads validated by type
- [ ] Uploaded files renamed
- [ ] Upload directory outside webroot
- [ ] No user input in include/require

### Configuration

- [ ] display_errors = Off in production
- [ ] error_log enabled
- [ ] Dangerous functions disabled
- [ ] open_basedir set
- [ ] HTTPS enforced

### Dependencies

- [ ] Dependencies regularly updated
- [ ] Known vulnerabilities checked (composer audit)
- [ ] No abandoned packages

## Common Pitfalls
1. **Treating security as a one-time task** — Security is ongoing. New vulnerabilities are discovered daily, and dependencies must be regularly audited.
2. **Relying solely on automated scanners** — Scanners catch known patterns but miss business logic flaws. Combine automated tools with manual code review.

## Best Practices
1. **Integrate security into CI/CD** — Run `composer audit`, static analysis (PHPStan, Psalm), and OWASP ZAP scans on every pull request.
2. **Maintain a security headers checklist** — Verify all security headers are present on every deployment using automated tests.

## Summary
- A comprehensive security checklist covers headers, configuration, authentication, input validation, and output encoding.
- Automate security checks in CI/CD with `composer audit`, static analysis, and vulnerability scanners.
- Security is an ongoing process — schedule regular audits and dependency updates.

## Code Examples

**Automated PHP configuration security audit function — checks critical php.ini settings**

```php
<?php
// Automated security check script
function auditPhpConfig(): array {
    $issues = [];
    
    if (ini_get('display_errors')) {
        $issues[] = 'display_errors is On';
    }
    if (!ini_get('log_errors')) {
        $issues[] = 'log_errors is Off';
    }
    if (ini_get('expose_php')) {
        $issues[] = 'expose_php is On';
    }
    if (!ini_get('session.cookie_httponly')) {
        $issues[] = 'session.cookie_httponly is Off';
    }
    
    return $issues;
}
```


## Resources

- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) — OWASP comprehensive web security testing guide

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*