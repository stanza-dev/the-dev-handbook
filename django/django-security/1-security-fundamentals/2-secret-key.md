---
source_course: "django-security"
source_lesson: "django-security-secret-key"
---

# Managing Secret Keys

## Introduction

The SECRET_KEY is Django's cryptographic backbone—it signs cookies, generates CSRF tokens, and protects sessions. A compromised secret key means your entire application's security is compromised.

## Key Concepts

**SECRET_KEY**: A random string used by Django for cryptographic signing. Must be unique per deployment and kept confidential.

**Cryptographic Signing**: Using the secret key to create tamper-proof tokens and cookies.

**Key Rotation**: The process of changing the secret key while maintaining service.

## Real World Context

Leaked secret keys enable attackers to:
- Forge session cookies and impersonate any user
- Create valid CSRF tokens to bypass protection
- Decrypt signed data
- Execute remote code if using pickle serialization

## Deep Dive

### Generating a Secure Key

```python
# Using Django's utility
from django.core.management.utils import get_random_secret_key
print(get_random_secret_key())  # 50-char random string

# Using Python secrets
import secrets
print(secrets.token_urlsafe(50))
```

### Environment Variables

```python
# settings.py
import os

SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')

if not SECRET_KEY:
    raise ValueError('DJANGO_SECRET_KEY environment variable not set')
```

```bash
# .env (never commit this!)
DJANGO_SECRET_KEY=your-super-secret-key-here
```

### Key Rotation

```python
# Use SECRET_KEY_FALLBACKS for rotation (Django 4.1+)
SECRET_KEY = os.environ['DJANGO_SECRET_KEY_NEW']
SECRET_KEY_FALLBACKS = [
    os.environ['DJANGO_SECRET_KEY_OLD'],
]
```

## Common Pitfalls

1. **Committing to version control**: Never commit SECRET_KEY. Use environment variables.
2. **Using the default key**: Django's auto-generated key is for development only.
3. **Sharing across environments**: Each environment (dev, staging, prod) needs its own key.

## Best Practices

1. **Use environment variables**: Never hardcode secrets.
2. **Use a secrets manager**: AWS Secrets Manager, HashiCorp Vault, etc.
3. **Rotate periodically**: Change keys annually or after team changes.
4. **Different keys per environment**: Never share keys between environments.

## Summary

The SECRET_KEY is critical infrastructure. Store it in environment variables, never commit it to version control, use unique keys per environment, and rotate periodically. A compromised key compromises everything.

## Resources

- [Django Settings](https://docs.djangoproject.com/en/6.0/ref/settings/#secret-key) — SECRET_KEY reference

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*