---
source_course: "django-security"
source_lesson: "django-security-allowed-hosts"
---

# ALLOWED_HOSTS and Host Header Attacks

## Introduction

The Host header tells Django which domain was requested. Attackers can manipulate this header to poison caches, steal credentials, or bypass security controls. ALLOWED_HOSTS is your defense.

## Key Concepts

**Host Header**: HTTP header indicating the domain the client is requesting.

**Host Header Injection**: Attack where malicious Host values trick the application.

**ALLOWED_HOSTS**: Django setting that whitelists valid Host header values.

## Real World Context

Host header attacks can:
- Poison password reset links (send reset emails with attacker's domain)
- Bypass access controls based on domain
- Cache poisoning in CDNs
- SSRF (Server-Side Request Forgery) exploitation

## Deep Dive

### Configuring ALLOWED_HOSTS

```python
# settings.py

# Production - explicit domains only
ALLOWED_HOSTS = ['example.com', 'www.example.com']

# With subdomain wildcard
ALLOWED_HOSTS = ['.example.com']  # Matches any subdomain

# NEVER in production
ALLOWED_HOSTS = ['*']  # Accepts ANY host - dangerous!
```

### Environment-Based Configuration

```python
import os

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Development fallback
if DEBUG:
    ALLOWED_HOSTS = ['localhost', '127.0.0.1', '[::1]']
```

### Password Reset Attack Example

```python
# Vulnerable code
def password_reset(request):
    token = generate_token(user)
    # Attacker sets Host: evil.com
    reset_url = f"http://{request.get_host()}/reset/{token}"
    send_email(user.email, reset_url)  # Link points to evil.com!
```

```python
# Safe code
def password_reset(request):
    token = generate_token(user)
    # Use explicit domain
    reset_url = f"https://example.com/reset/{token}"
    send_email(user.email, reset_url)
```

## Common Pitfalls

1. **Using ALLOWED_HOSTS = ['*']**: Disables protection entirely.
2. **Forgetting www variant**: Users on www.example.com get 400 errors.
3. **Using request.get_host() in sensitive places**: Can return attacker-controlled values.

## Best Practices

1. **Explicit domains only**: List exact domains, avoid wildcards.
2. **Use absolute URLs for sensitive links**: Don't rely on Host header.
3. **Configure CSRF_TRUSTED_ORIGINS**: Required for HTTPS cross-origin requests.

## Summary

ALLOWED_HOSTS protects against Host header injection attacks. Always configure it explicitly in production, never use '*', and avoid using request.get_host() for security-sensitive URLs like password resets.

## Resources

- [ALLOWED_HOSTS](https://docs.djangoproject.com/en/6.0/ref/settings/#allowed-hosts) — ALLOWED_HOSTS setting reference

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*