---
source_course: "php-security"
source_lesson: "php-security-secure-configuration"
---

# Secure PHP Configuration

## Introduction
Proper PHP configuration is your first line of defense. Many attacks exploit misconfigured servers.

## Key Concepts
- **display_errors**: Must be `Off` in production to prevent leaking file paths and database details.
- **expose_php**: Should be `Off` to hide PHP version from HTTP headers.
- **open_basedir**: Restricts file operations to specified directories, limiting damage from directory traversal.
- **disable_functions**: Blocks dangerous functions like `exec()`, `system()`, and `passthru()`.

## Real World Context
Misconfigured PHP installations are one of the most common attack vectors. A single `display_errors = On` in production can reveal database credentials, file system paths, and internal logic to attackers performing reconnaissance.

## Deep Dive
### Intro

Proper PHP configuration is your first line of defense. Many attacks exploit misconfigured servers.

### Critical php.ini settings

```ini
; Disable dangerous functions
disable_functions = exec,passthru,shell_exec,system,proc_open,popen

; Limit file operations
open_basedir = /var/www/html:/tmp

; Hide PHP version
expose_php = Off

; Error handling (production)
display_errors = Off
display_startup_errors = Off
log_errors = On
error_log = /var/log/php/error.log
error_reporting = E_ALL

; Session security
session.cookie_httponly = On
session.cookie_secure = On
session.cookie_samesite = Strict
session.use_strict_mode = On
session.use_only_cookies = On

; Upload limits
file_uploads = On
upload_max_filesize = 10M
max_file_uploads = 5

; Resource limits
max_execution_time = 30
max_input_time = 60
memory_limit = 128M
post_max_size = 10M
```

### Runtime configuration

```php
<?php
// Set at runtime (if not locked in php.ini)
ini_set('display_errors', '0');
ini_set('log_errors', '1');

// Verify settings
if (ini_get('display_errors')) {
    throw new RuntimeException('display_errors must be disabled in production');
}
```

### Environment-based configuration

```php
<?php
// config/security.php
return match($_ENV['APP_ENV'] ?? 'production') {
    'development' => [
        'display_errors' => true,
        'debug' => true,
    ],
    'production' => [
        'display_errors' => false,
        'debug' => false,
        'force_https' => true,
    ],
    default => throw new RuntimeException('Unknown environment'),
};
```

### Security headers

```php
<?php
function setSecurityHeaders(): void {
    // Prevent clickjacking
    header('X-Frame-Options: DENY');
    
    // Prevent MIME sniffing
    header('X-Content-Type-Options: nosniff');
    
    // XSS Protection (legacy browsers)
    header('X-XSS-Protection: 1; mode=block');
    
    // Content Security Policy
    header("Content-Security-Policy: default-src 'self'; script-src 'self'");
    
    // Force HTTPS
    header('Strict-Transport-Security: max-age=31536000; includeSubDomains');
    
    // Referrer Policy
    header('Referrer-Policy: strict-origin-when-cross-origin');
}
```

## Common Pitfalls
1. **Copying development php.ini to production** — Development settings like `display_errors=On` and `error_reporting=E_ALL` are dangerous in production.
2. **Forgetting session configuration** — Insecure session settings (`session.cookie_httponly=0`) allow session hijacking via XSS.

## Best Practices
1. **Maintain separate php.ini files** — Use environment-specific configurations for development, staging, and production.
2. **Audit php.ini regularly** — Use `php --ini` and `phpinfo()` (temporarily, never in production) to verify all security-relevant directives.

## Summary
- Secure PHP configuration starts with disabling `display_errors`, `expose_php`, and dangerous functions.
- Use `open_basedir` and `disable_functions` to limit the attack surface.
- Always maintain separate configurations for each environment.

## Code Examples

**Comprehensive security configuration class**

```php
<?php
declare(strict_types=1);

// Security configuration class
final class SecurityConfig {
    private static bool $initialized = false;
    
    public static function initialize(): void {
        if (self::$initialized) return;
        
        // Production checks
        if ($_ENV['APP_ENV'] === 'production') {
            self::enforceProductionSettings();
        }
        
        // Set security headers
        self::setHeaders();
        
        // Configure session
        self::configureSession();
        
        self::$initialized = true;
    }
    
    private static function enforceProductionSettings(): void {
        ini_set('display_errors', '0');
        ini_set('log_errors', '1');
        
        // Force HTTPS
        if (empty($_SERVER['HTTPS']) || $_SERVER['HTTPS'] === 'off') {
            header('Location: https://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'], true, 301);
            exit;
        }
    }
    
    private static function setHeaders(): void {
        header('X-Frame-Options: DENY');
        header('X-Content-Type-Options: nosniff');
        header('Referrer-Policy: strict-origin-when-cross-origin');
        
        if ($_ENV['APP_ENV'] === 'production') {
            header('Strict-Transport-Security: max-age=31536000');
        }
    }
    
    private static function configureSession(): void {
        ini_set('session.cookie_httponly', '1');
        ini_set('session.cookie_secure', '1');
        ini_set('session.cookie_samesite', 'Strict');
        ini_set('session.use_strict_mode', '1');
    }
}

// Call early in bootstrap
SecurityConfig::initialize();
?>
```


## Resources

- [Security Configuration](https://www.php.net/manual/en/security.general.php) — PHP security general considerations

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*