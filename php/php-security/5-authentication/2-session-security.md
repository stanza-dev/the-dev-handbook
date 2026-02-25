---
source_course: "php-security"
source_lesson: "php-security-session-security"
---

# Secure Session Management

## Introduction
Sessions maintain user state across requests. Poor session management leads to account hijacking.

## Key Concepts
- **Session Fixation**: Attack where the attacker sets a known session ID before the victim logs in.
- **Session Hijacking**: Stealing an active session ID via XSS, network sniffing, or log exposure.
- **session_regenerate_id(true)**: Replaces the session ID and deletes the old session file, preventing fixation.
- **Secure Cookie Flags**: `httponly`, `secure`, `samesite` attributes protect session cookies from theft.

## Real World Context
Session attacks are among the most common web vulnerabilities. The Firesheep tool (2010) demonstrated how trivially session IDs could be stolen on public WiFi, forcing the industry to adopt HTTPS everywhere. Always regenerate session IDs after authentication state changes.

## Deep Dive
### Intro

Sessions maintain user state across requests. Poor session management leads to account hijacking.

### Secure session configuration

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

### Session fixation prevention

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

### Session validation

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

### Secure logout

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

### Remember me tokens

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

## Common Pitfalls
1. **Forgetting to regenerate session ID after login** — Without `session_regenerate_id(true)`, an attacker who set the session ID before login can hijack the authenticated session.
2. **Storing sensitive data in sessions without encryption** — Session files on shared hosting can be readable by other users. Encrypt sensitive session data or use a private session handler.

## Best Practices
1. **Call `session_regenerate_id(true)` after every privilege change** — Login, logout, password change, and role elevation should all trigger session ID regeneration.
2. **Configure strict session settings** — Set `session.cookie_httponly=1`, `session.cookie_secure=1`, `session.use_strict_mode=1`, and `session.cookie_samesite=Lax`.

## Summary
- Prevent session fixation by calling `session_regenerate_id(true)` after every authentication state change.
- Configure session cookies with `httponly`, `secure`, and `samesite` flags.
- Use strict session mode and set appropriate session lifetime limits.

## Resources

- [Session Security](https://www.php.net/manual/en/session.security.php) — PHP session security best practices

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*