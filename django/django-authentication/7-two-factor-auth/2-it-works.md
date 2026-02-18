---
source_course: "django-authentication"
source_lesson: "django-authentication-totp-how-it-works"
---

# How TOTP Works

## Introduction

Understand the algorithm behind authenticator apps like Google Authenticator.

## Key Concepts

**TOTP**: Time-based One-Time Password.

**Shared Secret**: Key shared between app and server.

## Deep Dive

### TOTP Algorithm

```
1. Shared secret established during setup
2. Current time divided into 30-second windows
3. HMAC-SHA1(secret, time_window) generates hash
4. 6 digits extracted from hash

Code = HOTP(secret, floor(time/30))
```

### Why It's Secure

```
- Secret never transmitted after setup
- Codes valid for only 30 seconds
- Codes can't be reused
- Time window prevents replay attacks
```

### QR Code Setup

```python
import pyotp
import qrcode

# Generate secret
secret = pyotp.random_base32()

# Create provisioning URI
totp = pyotp.TOTP(secret)
uri = totp.provisioning_uri(
    name=user.email,
    issuer_name='MyApp'
)

# Generate QR code
qr = qrcode.make(uri)
```

## Best Practices

1. **Backup codes**: Provide recovery options.
2. **Allow time drift**: Accept ±1 time window.

## Summary

TOTP combines a shared secret with time to generate codes. Codes are valid for 30 seconds. QR codes simplify setup.

## Resources

- [RFC 6238 TOTP](https://datatracker.ietf.org/doc/html/rfc6238) — TOTP RFC

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*