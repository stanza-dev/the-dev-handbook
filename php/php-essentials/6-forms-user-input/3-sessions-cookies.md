---
source_course: "php-essentials"
source_lesson: "php-essentials-sessions-cookies"
---

# Sessions and Cookies

## Introduction

HTTP is a stateless protocol — each request is independent and carries no memory of previous interactions. But web applications need state: shopping carts, login sessions, and user preferences must persist across page loads. Cookies and sessions solve this problem in complementary ways. This lesson covers both mechanisms and when to use each.

## Key Concepts

- **Cookie**: A small piece of data (up to 4KB) stored on the client's browser and sent with every request to the server.
- **Session**: Server-side storage identified by a session ID cookie. Data lives on the server, only the ID travels in the browser.
- **`session_start()`**: Initializes or resumes a session. Must be called before any output.
- **`session_regenerate_id()`**: Creates a new session ID while preserving session data, essential after login to prevent session fixation.
- **Flash message**: A session value that is displayed once and then automatically deleted.

## Real World Context

Every authenticated web application uses sessions to track logged-in users. E-commerce sites use sessions for shopping carts and cookies for "remember me" functionality. Understanding the difference between client-side cookies and server-side sessions is critical for building secure applications. Getting this wrong leads to session hijacking, fixation attacks, and data leakage.

## Deep Dive

### Cookies

Cookies are set with `setcookie()` and read from `$_COOKIE`. They must be set before any HTML output:

```php
<?php
// Set a cookie (must be before any output)
setcookie('username', 'John', [
    'expires' => time() + (86400 * 30), // 30 days
    'path' => '/',
    'domain' => '',
    'secure' => true,      // HTTPS only
    'httponly' => true,    // No JavaScript access
    'samesite' => 'Strict' // CSRF protection
]);

// Read a cookie
$username = $_COOKIE['username'] ?? 'Guest';

// Delete a cookie (set expiry in past)
setcookie('username', '', time() - 3600);
```

The array-based options syntax (PHP 7.3+) is recommended over positional arguments because it is self-documenting. Always set `httponly` and `secure` for security.

### Sessions

Sessions store data on the server and identify users by a session ID cookie:

```php
<?php
// Start session (must be before any output)
session_start();

// Set session data
$_SESSION['user_id'] = 123;
$_SESSION['username'] = 'John';
$_SESSION['is_admin'] = false;

// Read session data
$userId = $_SESSION['user_id'] ?? null;

// Check if session variable exists
if (isset($_SESSION['username'])) {
    echo "Hello, " . $_SESSION['username'];
}

// Remove specific session variable
unset($_SESSION['temp_data']);

// Destroy entire session (logout)
session_destroy();
```

`session_start()` must be called at the top of every script that needs session access, before any output is sent to the browser.

### Session Security

Secure your sessions with these configuration options and regeneration:

```php
<?php
// Secure session configuration (before session_start)
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_secure', 1);
ini_set('session.cookie_samesite', 'Strict');
ini_set('session.use_strict_mode', 1);

session_start();

// Regenerate ID after login (prevent session fixation)
if ($loginSuccessful) {
    session_regenerate_id(true);
    $_SESSION['user_id'] = $user->id;
}
```

Regenerating the session ID after authentication is critical. Without it, an attacker who knows the pre-login session ID retains access after login.

### Flash Messages

Flash messages are one-time notifications that display once and then disappear:

```php
<?php
session_start();

// Set flash message
function setFlash(string $type, string $message): void {
    $_SESSION['flash'][$type] = $message;
}

// Get and clear flash message
function getFlash(string $type): ?string {
    $message = $_SESSION['flash'][$type] ?? null;
    unset($_SESSION['flash'][$type]);
    return $message;
}

// Usage: after processing a form
setFlash('success', 'Account created successfully!');

// On next page load
if ($message = getFlash('success')) {
    echo "<div class='alert success'>$message</div>";
}
```

This pattern is used by every PHP framework for displaying success/error messages after redirects.

### Cookies vs Sessions

| Feature | Cookies | Sessions |
|---------|---------|----------|
| Storage | Client (browser) | Server |
| Security | Less secure (visible to client) | More secure (data on server) |
| Size limit | ~4KB | No practical limit |
| Lifetime | Configurable (days/months) | Browser session (default) |
| Use for | Preferences, "remember me" | Auth, shopping carts, sensitive data |

## Common Pitfalls

1. **Sending output before `session_start()` or `setcookie()`** — Both functions send HTTP headers, which must come before any HTML, whitespace, or `echo` statements. Enable output buffering or move session/cookie code to the very top of your script.
2. **Not regenerating the session ID after login** — This leaves your application vulnerable to session fixation attacks where an attacker pre-sets the session ID.
3. **Storing sensitive data in cookies** — Cookies are stored on the client and can be read or modified. Never store passwords, tokens, or personal data in cookies. Use sessions for sensitive information.

## Best Practices

1. **Always configure session cookies as secure and httponly** — Set `secure`, `httponly`, and `samesite` attributes to protect against XSS and CSRF.
2. **Call `session_regenerate_id(true)` after authentication** — The `true` parameter deletes the old session file, preventing reuse.
3. **Implement proper session cleanup on logout** — Clear `$_SESSION`, delete the session cookie, and call `session_destroy()` to fully terminate the session.

## Summary

- HTTP is stateless; cookies and sessions add state across requests.
- Cookies store small data on the client; sessions store data on the server.
- Always call `session_start()` before any output and before accessing `$_SESSION`.
- Regenerate the session ID after login with `session_regenerate_id(true)` to prevent fixation.
- Use cookies for non-sensitive preferences and sessions for authentication and sensitive data.

## Code Examples

**Session-based authentication with login, logout, and auth-guard functions demonstrating session_regenerate_id and proper cleanup**

```php
<?php
// Simple authentication with sessions
session_start();

function login(string $email, string $password): bool {
    // Verify credentials (in real app, check database)
    $user = findUserByEmail($email);
    
    if ($user && password_verify($password, $user['password_hash'])) {
        // Regenerate session ID for security
        session_regenerate_id(true);
        
        // Store user data in session
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['user_email'] = $user['email'];
        $_SESSION['logged_in_at'] = time();
        
        return true;
    }
    return false;
}

function isLoggedIn(): bool {
    return isset($_SESSION['user_id']);
}

function logout(): void {
    $_SESSION = [];
    
    // Delete session cookie
    if (isset($_COOKIE[session_name()])) {
        setcookie(session_name(), '', time() - 42000);
    }
    
    session_destroy();
}

function requireAuth(): void {
    if (!isLoggedIn()) {
        header('Location: /login.php');
        exit;
    }
}
?>
```


## Resources

- [PHP Sessions](https://www.php.net/manual/en/book.session.php) — Complete session handling documentation
- [Session Security](https://www.php.net/manual/en/session.security.php) — Best practices for secure session handling

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*