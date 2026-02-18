---
source_course: "php-security"
source_lesson: "php-security-threat-landscape"
---

# Understanding Web Security Threats

Web applications face constant security threats. Understanding these threats is the first step to defending against them.

## OWASP Top 10 for PHP

The Open Web Application Security Project (OWASP) identifies the most critical web security risks:

1. **Injection** (SQL, Command, LDAP)
2. **Broken Authentication**
3. **Sensitive Data Exposure**
4. **XML External Entities (XXE)**
5. **Broken Access Control**
6. **Security Misconfiguration**
7. **Cross-Site Scripting (XSS)**
8. **Insecure Deserialization**
9. **Using Components with Known Vulnerabilities**
10. **Insufficient Logging & Monitoring**

## Attack Vectors

### User Input
```php
<?php
// Every user input is potentially malicious
$_GET['search'];      // URL parameters
$_POST['username'];   // Form data
$_COOKIE['session'];  // Cookies
$_FILES['upload'];    // File uploads
$_SERVER['HTTP_*'];   // HTTP headers
file_get_contents('php://input');  // Raw POST body
```

### Trust Boundaries

```
[Untrusted Zone]        [Trust Boundary]         [Trusted Zone]

  User Input  ────────────► Validation ────────────► Application
  External APIs ──────────► Sanitization ─────────► Database
  File Uploads ───────────► Authorization ────────► File System
```

## Defense in Depth

Never rely on a single security measure:

```php
<?php
// Layer 1: Input Validation
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);

// Layer 2: Parameterized Query (SQL Injection prevention)
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);

// Layer 3: Access Control
if (!$user->canViewData($result)) {
    throw new ForbiddenException();
}

// Layer 4: Output Encoding (XSS prevention)
echo htmlspecialchars($result['name'], ENT_QUOTES, 'UTF-8');
```

## Security Principles

| Principle | Description |
|-----------|-------------|
| Least Privilege | Grant minimum permissions needed |
| Defense in Depth | Multiple layers of security |
| Fail Securely | Errors should deny access, not grant it |
| Never Trust Input | Validate everything from users |
| Keep It Simple | Complex code has more vulnerabilities |

## Resources

- [PHP Security Manual](https://www.php.net/manual/en/security.php) — Official PHP security documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*