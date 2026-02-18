---
source_course: "django-security"
source_lesson: "django-security-mfa"
---

# Multi-Factor Authentication

## Introduction

Passwords alone aren't enough. MFA adds a second verification factor—something you have (phone, security key) in addition to something you know (password).

## Key Concepts

**TOTP**: Time-based One-Time Password—apps like Google Authenticator.

**U2F/WebAuthn**: Hardware security keys like YubiKey.

**SMS/Email OTP**: One-time codes sent via text or email (less secure).

## Deep Dive

### Using django-two-factor-auth

```python
# pip install django-two-factor-auth

# settings.py
INSTALLED_APPS = [
    'django_otp',
    'django_otp.plugins.otp_static',
    'django_otp.plugins.otp_totp',
    'two_factor',
    ...
]

MIDDLEWARE = [
    'django_otp.middleware.OTPMiddleware',
    ...
]

LOGIN_URL = 'two_factor:login'
```

### Enforcing MFA for Admins

```python
from django_otp.decorators import otp_required

@otp_required
def admin_view(request):
    # Only accessible after MFA verification
    pass
```

### WebAuthn Integration

```python
# pip install django-webauthn

# Modern passwordless/MFA with security keys
from webauthn import verify_authentication_response
```

## Best Practices

1. **Require MFA for privileged users**: Admins, finance access.
2. **Provide backup codes**: In case of lost authenticator.
3. **Prefer TOTP/WebAuthn**: Over SMS (SIM swapping attacks).
4. **Grace period for setup**: Let users enable MFA after initial registration.

## Summary

MFA significantly improves security. Use django-two-factor-auth for TOTP, prefer hardware keys for highest security, and always provide backup codes for account recovery.

## Resources

- [django-two-factor-auth](https://django-two-factor-auth.readthedocs.io/) — Django 2FA documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*