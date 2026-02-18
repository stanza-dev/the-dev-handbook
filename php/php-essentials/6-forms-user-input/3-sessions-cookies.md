---
source_course: "php-essentials"
source_lesson: "php-essentials-sessions-cookies"
---

# Sessions and Cookies

HTTP is stateless - sessions and cookies help maintain state across requests.

## Cookies

Stored on the client's browser:

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

## Sessions

Server-side storage, more secure for sensitive data:

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

## Session Security

```php
<?php
// Secure session configuration (before session_start)
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_secure', 1);
ini_set('session.cookie_samesite', 'Strict');
ini_set('session.use_strict_mode', 1);

session_start();

// Regenerate ID after login (prevent fixation)
if ($loginSuccessful) {
    session_regenerate_id(true);
    $_SESSION['user_id'] = $user->id;
}
```

## Flash Messages

One-time messages that disappear after display:

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

// Usage
setFlash('success', 'Account created successfully!');

// On next page load
if ($message = getFlash('success')) {
    echo "<div class='alert success'>$message</div>";
}
```

## Cookies vs Sessions

| Feature | Cookies | Sessions |
|---------|---------|----------|
| Storage | Client | Server |
| Security | Less secure | More secure |
| Size limit | ~4KB | No limit |
| Lifetime | Configurable | Browser session |
| Use for | Preferences, "remember me" | Auth, sensitive data |

## Code Examples

**Session-based authentication functions**

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