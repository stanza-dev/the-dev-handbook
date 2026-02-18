---
source_course: "php-security"
source_lesson: "php-security-session-security"
---

# Secure Session Management

Sessions maintain user state across requests. Poor session management leads to account hijacking.

## Secure Session Configuration

```php
<?php
// Before session_start()
ini_set('session.cookie_httponly', '1');    // No JS access
ini_set('session.cookie_secure', '1');       // HTTPS only
ini_set('session.cookie_samesite', 'Strict'); // No cross-site
ini_set('session.use_strict_mode', '1');     // Reject unknown IDs
ini_set('session.use_only_cookies', '1');    // No URL sessions

session_start();
```

## Session Fixation Prevention

```php
<?php
// Regenerate ID after privilege change
function login(User $user): void {
    // Regenerate session ID to prevent fixation
    session_regenerate_id(true);  // true = delete old session
    
    $_SESSION['user_id'] = $user->id;
    $_SESSION['logged_in_at'] = time();
    $_SESSION['ip'] = $_SERVER['REMOTE_ADDR'];
    $_SESSION['user_agent'] = $_SERVER['HTTP_USER_AGENT'];
}
```

## Session Validation

```php
<?php
function validateSession(): bool {
    if (!isset($_SESSION['user_id'])) {
        return false;
    }
    
    // Check for session hijacking indicators
    if ($_SESSION['ip'] !== $_SERVER['REMOTE_ADDR']) {
        // IP changed - suspicious, but could be legitimate
        // Log and optionally invalidate
    }
    
    if ($_SESSION['user_agent'] !== $_SERVER['HTTP_USER_AGENT']) {
        // User agent changed - likely hijacking
        session_destroy();
        return false;
    }
    
    // Check session age
    $maxAge = 3600; // 1 hour
    if (time() - $_SESSION['logged_in_at'] > $maxAge) {
        session_destroy();
        return false;
    }
    
    return true;
}
```

## Secure Logout

```php
<?php
function logout(): void {
    // Clear session data
    $_SESSION = [];
    
    // Delete session cookie
    if (ini_get('session.use_cookies')) {
        $params = session_get_cookie_params();
        setcookie(
            session_name(),
            '',
            time() - 42000,
            $params['path'],
            $params['domain'],
            $params['secure'],
            $params['httponly']
        );
    }
    
    // Destroy session
    session_destroy();
}
```

## Remember Me Tokens

```php
<?php
function createRememberToken(int $userId): string {
    $selector = bin2hex(random_bytes(16));
    $validator = bin2hex(random_bytes(32));
    $hashedValidator = hash('sha256', $validator);
    
    // Store selector + hashed validator in database
    $stmt = $pdo->prepare(
        'INSERT INTO remember_tokens (user_id, selector, hashed_validator, expires_at) 
         VALUES (:user_id, :selector, :hashed_validator, :expires_at)'
    );
    $stmt->execute([
        'user_id' => $userId,
        'selector' => $selector,
        'hashed_validator' => $hashedValidator,
        'expires_at' => date('Y-m-d H:i:s', time() + 86400 * 30),
    ]);
    
    // Return selector:validator to set in cookie
    return $selector . ':' . $validator;
}
```

## Resources

- [Session Security](https://www.php.net/manual/en/session.security.php) — PHP session security best practices

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*