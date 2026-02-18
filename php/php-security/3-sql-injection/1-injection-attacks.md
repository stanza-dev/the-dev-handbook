---
source_course: "php-security"
source_lesson: "php-security-sql-injection-attacks"
---

# Understanding SQL Injection

SQL injection is one of the most dangerous and common web vulnerabilities. It allows attackers to execute arbitrary SQL commands.

## How SQL Injection Works

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

## Types of SQL Injection

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

## Real Attack Examples

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

## The Damage

- **Data theft**: Steal entire databases
- **Data modification**: Alter records, prices, permissions
- **Data deletion**: Drop tables, truncate data
- **Authentication bypass**: Login as any user
- **Remote code execution**: Some databases allow OS commands

## Resources

- [SQL Injection Prevention](https://www.php.net/manual/en/security.database.sql-injection.php) — Official PHP SQL injection prevention guide

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*