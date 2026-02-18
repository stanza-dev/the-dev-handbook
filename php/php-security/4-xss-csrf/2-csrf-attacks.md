---
source_course: "php-security"
source_lesson: "php-security-csrf-attacks"
---

# Cross-Site Request Forgery (CSRF)

CSRF tricks users into performing actions they didn't intend, using their authenticated session.

## How CSRF Works

```html
<!-- Evil website contains: -->
<img src="https://bank.com/transfer?to=attacker&amount=10000">

<!-- Or a hidden form that auto-submits: -->
<form action="https://bank.com/transfer" method="POST" id="csrf">
  <input type="hidden" name="to" value="attacker">
  <input type="hidden" name="amount" value="10000">
</form>
<script>document.getElementById('csrf').submit();</script>
```

If the user is logged into bank.com, the request includes their session cookie automatically!

## Protection: CSRF Tokens

```php
<?php
session_start();

// Generate token (once per session)
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

function getCsrfToken(): string {
    return $_SESSION['csrf_token'];
}

function validateCsrfToken(string $token): bool {
    return hash_equals($_SESSION['csrf_token'] ?? '', $token);
}
```

## Using CSRF Tokens in Forms

```php
<?php
// In the form
?>
<form method="POST" action="/transfer">
    <input type="hidden" name="csrf_token" value="<?= htmlspecialchars(getCsrfToken()) ?>">
    <input type="text" name="amount">
    <button type="submit">Transfer</button>
</form>

<?php
// Processing the form
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $token = $_POST['csrf_token'] ?? '';
    
    if (!validateCsrfToken($token)) {
        http_response_code(403);
        die('Invalid CSRF token');
    }
    
    // Process the valid request
}
```

## Double Submit Cookie Pattern

```php
<?php
// Set token in cookie AND form
$token = bin2hex(random_bytes(32));
setcookie('csrf_token', $token, [
    'httponly' => false,  // JS needs to read it
    'samesite' => 'Strict',
    'secure' => true,
]);

// Verify both match
function validateDoubleSubmit(string $formToken): bool {
    $cookieToken = $_COOKIE['csrf_token'] ?? '';
    return hash_equals($cookieToken, $formToken);
}
```

## SameSite Cookies

```php
<?php
// Modern defense: SameSite cookies
setcookie('session_id', $sessionId, [
    'expires' => time() + 3600,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Strict'  // Not sent on cross-site requests
]);

// Or in session config
ini_set('session.cookie_samesite', 'Strict');
```

## Code Examples

**Complete CSRF protection class**

```php
<?php
declare(strict_types=1);

// Complete CSRF protection class
class CsrfProtection {
    private const TOKEN_LENGTH = 32;
    private const TOKEN_NAME = 'csrf_token';
    
    public function __construct() {
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
    }
    
    public function getToken(): string {
        if (empty($_SESSION[self::TOKEN_NAME])) {
            $_SESSION[self::TOKEN_NAME] = bin2hex(random_bytes(self::TOKEN_LENGTH));
        }
        return $_SESSION[self::TOKEN_NAME];
    }
    
    public function getTokenField(): string {
        $token = htmlspecialchars($this->getToken(), ENT_QUOTES, 'UTF-8');
        return sprintf(
            '<input type="hidden" name="%s" value="%s">',
            self::TOKEN_NAME,
            $token
        );
    }
    
    public function validate(?string $token = null): bool {
        $token ??= $_POST[self::TOKEN_NAME] ?? $_SERVER['HTTP_X_CSRF_TOKEN'] ?? '';
        $sessionToken = $_SESSION[self::TOKEN_NAME] ?? '';
        
        if (empty($sessionToken) || empty($token)) {
            return false;
        }
        
        return hash_equals($sessionToken, $token);
    }
    
    public function validateOrFail(): void {
        if (!$this->validate()) {
            http_response_code(403);
            throw new RuntimeException('CSRF validation failed');
        }
    }
    
    public function regenerate(): string {
        $_SESSION[self::TOKEN_NAME] = bin2hex(random_bytes(self::TOKEN_LENGTH));
        return $_SESSION[self::TOKEN_NAME];
    }
}

// Usage
$csrf = new CsrfProtection();

// In form
echo '<form method="POST">';
echo $csrf->getTokenField();
echo '<button type="submit">Submit</button></form>';

// On submission
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $csrf->validateOrFail();
    // Process form...
}
?>
```


---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*