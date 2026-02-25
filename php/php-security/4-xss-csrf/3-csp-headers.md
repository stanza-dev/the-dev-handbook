---
source_course: "php-security"
source_lesson: "php-security-csp-headers"
---

# Content Security Policy (CSP)

## Introduction
CSP is a powerful defense against XSS that tells browsers which resources are allowed to load and execute.

## Key Concepts
- **Content Security Policy (CSP)**: HTTP header that controls which resources the browser is allowed to load.
- **CSP Nonce**: A unique per-request token (`nonce-<random>`) that allows specific inline scripts while blocking injected ones.
- **Report-Only Mode**: `Content-Security-Policy-Report-Only` logs violations without blocking, useful for gradual CSP rollout.
- **CSP Directives**: `script-src`, `style-src`, `img-src`, `connect-src`, `default-src` — each controls a resource type.

## Real World Context
GitHub, Google, and Stripe all use strict CSP policies to mitigate XSS even when code-level defenses fail. Google reported that CSP prevented exploitation of several XSS bugs that were discovered after deployment. Start with Report-Only mode to avoid breaking your site during rollout.

## Deep Dive
### Intro

CSP is a powerful defense against XSS that tells browsers which resources are allowed to load and execute.

### Basic csp header

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

### Granular csp directives

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

### Using nonces for inline scripts

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

### Csp for apis

```php
<?php
// APIs should have very restrictive CSP
function setApiCSP(): void
{
    header("Content-Security-Policy: default-src 'none'; frame-ancestors 'none'");
    header('X-Content-Type-Options: nosniff');
}
```

### Report-only mode (testing)

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

### Common csp mistakes

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

## Common Pitfalls
1. **Using `unsafe-inline` in `script-src`** — This completely defeats CSP's XSS protection. Use nonce-based or hash-based policies instead.
2. **Deploying CSP without Report-Only testing** — A strict CSP can break third-party scripts, analytics, and CDN resources. Always test in Report-Only mode first.

## Best Practices
1. **Start with a strict nonce-based policy** — `script-src 'nonce-{random}' 'strict-dynamic'` provides strong XSS protection while allowing trusted scripts to load dependencies.
2. **Set up CSP violation reporting** — Use the `report-uri` or `report-to` directive to collect violation reports and identify blocked resources.

## Summary
- CSP is a critical defense-in-depth layer that prevents XSS exploitation even when code-level defenses fail.
- Use nonce-based policies instead of `unsafe-inline` to allow legitimate inline scripts.
- Always deploy new CSP policies in Report-Only mode first to catch breaking changes.

## Resources

- [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) — MDN CSP documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*