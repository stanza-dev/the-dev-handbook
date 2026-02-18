---
source_course: "php-security"
source_lesson: "php-security-security-checklist"
---

# Security Audit Checklist

Use this checklist to audit your PHP applications for common vulnerabilities.

## Input Handling

- [ ] All user input is validated
- [ ] Whitelist validation where possible
- [ ] Type declarations (strict_types)
- [ ] No raw $_GET, $_POST used in SQL
- [ ] No raw user input echoed to page

## SQL Security

- [ ] All queries use prepared statements
- [ ] PDO with ATTR_EMULATE_PREPARES = false
- [ ] Table/column names validated against whitelist
- [ ] Error messages don't reveal database structure

## Output Security

- [ ] All output escaped with htmlspecialchars()
- [ ] Content-Security-Policy header set
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options header set

## Authentication

- [ ] Passwords hashed with password_hash()
- [ ] Session regenerated after login
- [ ] Secure session cookie settings
- [ ] Account lockout after failed attempts
- [ ] CSRF tokens on all forms

## Session Management

- [ ] HTTPOnly cookies
- [ ] Secure flag on cookies
- [ ] SameSite cookie attribute
- [ ] Session timeout implemented
- [ ] Session data validated

## File Handling

- [ ] File uploads validated by type
- [ ] Uploaded files renamed
- [ ] Upload directory outside webroot
- [ ] No user input in include/require

## Configuration

- [ ] display_errors = Off in production
- [ ] error_log enabled
- [ ] Dangerous functions disabled
- [ ] open_basedir set
- [ ] HTTPS enforced

## Dependencies

- [ ] Dependencies regularly updated
- [ ] Known vulnerabilities checked (composer audit)
- [ ] No abandoned packages

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*