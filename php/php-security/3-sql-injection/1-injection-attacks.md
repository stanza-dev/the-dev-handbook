---
source_course: "php-security"
source_lesson: "php-security-sql-injection-attacks"
---

# Understanding SQL Injection

## Introduction
SQL injection is one of the most dangerous and common web vulnerabilities. It allows attackers to execute arbitrary SQL commands.

## Key Concepts
- **Data theft**: Stealing entire databases including user credentials and financial data.
- **Data modification**: Altering records, prices, permissions, or admin status.
- **Authentication bypass**: Logging in as any user by manipulating SQL WHERE clauses.
- **Remote code execution**: Some database configurations allow executing OS commands via SQL.

## Real World Context
SQL injection has caused some of the largest data breaches in history. The 2008 Heartland Payment Systems breach exposed 130 million credit cards through SQL injection. Despite being well-understood, SQL injection remains in the OWASP Top 10 because developers continue to concatenate user input into queries.

## Deep Dive
### Intro

SQL injection is one of the most dangerous and common web vulnerabilities. It allows attackers to execute arbitrary SQL commands.

### How sql injection works

```php
<?php
// VULNERABLE CODE - Never do this!
$username = $_POST['username'];
$query = "SELECT * FROM users WHERE username = '$username'";

// Normal input: "john"
// Query: SELECT * FROM users WHERE username = 'john'

// Malicious input: "' OR '1'='1"
// Query: SELECT * FROM users WHERE username = '' OR '1'='1'
// This returns ALL users!

// Even worse: "'; DROP TABLE users; --"
// Query: SELECT * FROM users WHERE username = ''; DROP TABLE users; --'
// This DELETES the entire table!
```

### Types of sql injection

### 1. Classic (In-band) Injection
```php
<?php
// Attacker sees results directly
$id = $_GET['id'];  // Input: "1 UNION SELECT username, password FROM users"
$query = "SELECT name FROM products WHERE id = $id";
// Returns all usernames and passwords!
```

### 2. Blind Injection
```php
<?php
// Attacker infers data from behavior
$id = $_GET['id'];  // Input: "1 AND 1=1" vs "1 AND 1=2"
// Different responses reveal information
```

### 3. Time-Based Injection
```php
<?php
// Input: "1; SELECT SLEEP(5)--"
// If page takes 5 seconds, injection works
```

### Real attack examples

```php
<?php
// Login bypass
$username = "admin'--";
$password = "anything";
$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
// Becomes: SELECT * FROM users WHERE username='admin'--' AND password='anything'
// The -- comments out the password check!

// Data extraction
$id = "1 UNION SELECT credit_card, cvv, expiry FROM payments--";
$query = "SELECT name, price FROM products WHERE id = $id";
// Returns credit card data instead of product info!
```

### The damage

- **Data theft**: Steal entire databases
- **Data modification**: Alter records, prices, permissions
- **Data deletion**: Drop tables, truncate data
- **Authentication bypass**: Login as any user
- **Remote code execution**: Some databases allow OS commands

## Common Pitfalls
1. **Concatenating user input into SQL strings** — Even a single concatenated variable creates a SQL injection vulnerability. Always use parameterized queries.
2. **Relying on input sanitization instead of prepared statements** — Sanitization functions like `addslashes()` can be bypassed with multi-byte character attacks. Prepared statements are the only reliable defense.

## Best Practices
1. **Use prepared statements for every query** — Never concatenate user input into SQL. Use PDO with named or positional parameters for all database interactions.
2. **Configure PDO securely** — Set `ATTR_ERRMODE` to `EXCEPTION` and `ATTR_EMULATE_PREPARES` to `false` for maximum security.

## Summary
- SQL injection allows attackers to execute arbitrary SQL by injecting code through user input.
- The three main types are classic (in-band), blind, and time-based SQL injection.
- Prepared statements with parameterized queries are the definitive defense against SQL injection.

## Resources

- [SQL Injection Prevention](https://www.php.net/manual/en/security.database.sql-injection.php) — Official PHP SQL injection prevention guide

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*