---
source_course: "django-security"
source_lesson: "django-security-https-settings"
---

# HTTPS and Secure Transport

## Introduction

HTTPS encrypts all communication between browser and server. Without it, passwords, cookies, and sensitive data travel in plain text, visible to anyone on the network.

## Key Concepts

**TLS/SSL**: Protocols that encrypt HTTP traffic (HTTPS = HTTP + TLS).

**HSTS**: HTTP Strict Transport Security—tells browsers to always use HTTPS.

**Secure Cookies**: Cookies that are only sent over HTTPS connections.



## Real World Context

In 2014, the Heartbleed bug exposed millions of HTTPS-protected sites. But even without such vulnerabilities, any application serving login forms or sensitive data over plain HTTP exposes credentials to anyone on the same network—coffee shop Wi-Fi, hotel networks, or compromised ISPs.

## Deep Dive

### Essential HTTPS Settings

```python
# settings.py (production)

# Redirect HTTP to HTTPS
SECURE_SSL_REDIRECT = True

# HSTS - instruct browsers to use HTTPS
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Secure cookies
SESSION_COOKIE_SECURE = True  # Session cookie only over HTTPS
CSRF_COOKIE_SECURE = True      # CSRF cookie only over HTTPS

# Behind a proxy (Nginx, load balancer)
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

### Understanding HSTS

```
First visit: HTTP → 301 Redirect → HTTPS
Subsequent: Browser goes directly to HTTPS (even if user types http://)
```

**HSTS Preload**: Submit your domain to browser's built-in HTTPS list.

### Handling Proxies

```python
# When behind Nginx/Load Balancer, SSL terminates at proxy
# Django sees HTTP, but actual connection is HTTPS

# Trust X-Forwarded-Proto header from proxy
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

## Common Pitfalls

1. **Starting with long HSTS**: Start with short duration (300 seconds) for testing.
2. **Forgetting subdomains**: HSTS with includeSubDomains affects ALL subdomains.
3. **Cookie theft on HTTP**: Without SECURE flag, cookies sent over HTTP.

## Best Practices

1. **Use HTTPS everywhere**: No exceptions for 'unimportant' pages.
2. **Enable HSTS**: Protects against SSL stripping attacks.
3. **Test before HSTS preload**: Preload is essentially permanent.
4. **All cookies secure**: Both session and CSRF cookies.

## Summary

HTTPS protects data in transit. Enable SECURE_SSL_REDIRECT, HSTS, and secure cookies. Start with short HSTS durations when testing, and consider HSTS preload for maximum security.

## Resources

- [SSL/HTTPS](https://docs.djangoproject.com/en/6.0/topics/security/#ssl-https) — Django SSL/HTTPS documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*