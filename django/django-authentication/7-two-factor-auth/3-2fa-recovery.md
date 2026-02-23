---
source_course: "django-authentication"
source_lesson: "django-authentication-2fa-recovery"
---

# 2FA Recovery Options

## Introduction

Users lose access to authenticators. Provide recovery options to prevent lockouts.

## Key Concepts

**Backup Codes**: One-time use recovery codes.

**Recovery Methods**: Alternative verification methods.

## Real World Context

A developer loses their phone and can no longer generate TOTP codes. Without backup codes, they are completely locked out and must contact support -- which for a solo project means resetting the database. Generating 10 single-use backup codes during 2FA setup is the standard safety net.

## Deep Dive

### Static Backup Codes

```python
from django_otp.plugins.otp_static.models import StaticDevice

def generate_backup_codes(user, count=10):
    device, created = StaticDevice.objects.get_or_create(
        user=user,
        name='backup'
    )
    
    # Generate codes
    codes = []
    for _ in range(count):
        token = StaticDevice.token_generator()
        device.token_set.create(token=token)
        codes.append(token)
    
    return codes
```

### Displaying Codes Once

```python
def show_backup_codes(request):
    codes = generate_backup_codes(request.user)
    # Show to user ONCE, then forget
    return render(request, 'backup_codes.html', {
        'codes': codes,
        'warning': 'Save these codes securely. They will not be shown again.'
    })
```

### Recovery Email Flow

```python
def request_recovery(request):
    # Send recovery link to verified email
    # Link temporarily bypasses 2FA
    # User must re-enable 2FA after recovery
```

## Common Pitfalls

1. **Not showing backup codes only once**: If backup codes are retrievable after initial display, a compromised session can be used to steal them. Generate, display, and then never show them again.
2. **Not invalidating used backup codes**: Each backup code must be single-use. After a code is used for recovery, delete it from the database immediately to prevent reuse.
3. **Skipping re-enrollment after recovery**: When a user recovers via backup code, their TOTP secret may be compromised. Force them to set up a new TOTP device and generate fresh backup codes.

## Best Practices

1. **Generate 10+ backup codes**: Users lose them.
2. **Show codes only once**: Security requirement.
3. **Require re-setup after recovery**: Ensures security.

## Summary

Provide backup codes for recovery. Show codes only once. Consider email-based recovery for enterprise apps.

## Resources

- [Static Tokens](https://django-two-factor-auth.readthedocs.io/en/stable/installation.html) — Backup codes

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*