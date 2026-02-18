---
source_course: "php-security"
source_lesson: "php-security-csp-headers"
---

# Content Security Policy (CSP)

CSP is a powerful defense against XSS that tells browsers which resources are allowed to load and execute.

## Basic CSP Header

```php
<?php
// Restrictive default policy
header("Content-Security-Policy: default-src 'self'");

// This prevents:
// - Inline scripts (<script>alert(1)</script>)
// - Inline styles
// - Loading resources from other domains
// - eval() and similar functions
```

## Granular CSP Directives

```php
<?php
function setCSP(): void
{
    $directives = [
        // Default fallback
        "default-src 'self'",
        
        // Scripts: own domain + specific CDN
        "script-src 'self' https://cdn.example.com",
        
        // Styles: own domain + inline (for frameworks)
        "style-src 'self' 'unsafe-inline'",
        
        // Images from anywhere
        "img-src 'self' data: https:",
        
        // Fonts from own domain and Google
        "font-src 'self' https://fonts.gstatic.com",
        
        // AJAX/Fetch to own domain only
        "connect-src 'self'",
        
        // Frames: none
        "frame-src 'none'",
        
        // Form submissions to own domain
        "form-action 'self'",
        
        // Don't allow embedding in frames
        "frame-ancestors 'none'",
        
        // Block mixed content
        "upgrade-insecure-requests",
    ];
    
    header('Content-Security-Policy: ' . implode('; ', $directives));
}
```

## Using Nonces for Inline Scripts

```php
<?php
// Generate nonce for this request
$nonce = base64_encode(random_bytes(16));

// Set CSP header with nonce
header("Content-Security-Policy: script-src 'nonce-$nonce'");

// Store for use in templates
$_REQUEST['csp_nonce'] = $nonce;
```

```html
<!-- Only scripts with matching nonce will execute -->
<script nonce="<?= htmlspecialchars($nonce) ?>">  // Allowed
    console.log('This runs');
</script>

<script>  // Blocked!
    console.log('This is blocked');
</script>
```

## CSP for APIs

```php
<?php
// APIs should have very restrictive CSP
function setApiCSP(): void
{
    header("Content-Security-Policy: default-src 'none'; frame-ancestors 'none'");
    header('X-Content-Type-Options: nosniff');
}
```

## Report-Only Mode (Testing)

```php
<?php
// Test CSP without blocking
header(
    "Content-Security-Policy-Report-Only: " .
    "default-src 'self'; " .
    "report-uri /csp-report"
);
```

```php
<?php
// /csp-report endpoint to collect violations
$report = json_decode(file_get_contents('php://input'), true);

if (isset($report['csp-report'])) {
    $violation = $report['csp-report'];
    error_log(sprintf(
        "CSP Violation: %s blocked %s from %s",
        $violation['violated-directive'],
        $violation['blocked-uri'],
        $violation['document-uri']
    ));
}
```

## Common CSP Mistakes

```php
<?php
// BAD: 'unsafe-inline' defeats XSS protection
header("Content-Security-Policy: script-src 'self' 'unsafe-inline'");

// BAD: 'unsafe-eval' allows eval() attacks
header("Content-Security-Policy: script-src 'self' 'unsafe-eval'");

// BAD: Wildcard allows any subdomain
header("Content-Security-Policy: script-src *.example.com");

// GOOD: Specific sources with nonces
header("Content-Security-Policy: script-src 'self' 'nonce-abc123'");
```

## Resources

- [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) — MDN CSP documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*