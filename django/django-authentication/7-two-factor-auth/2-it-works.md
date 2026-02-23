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

## Real World Context

When your compliance team requires MFA for admin access, TOTP is the most practical choice because it works offline. Unlike SMS codes (which are vulnerable to SIM-swapping) or email codes (which are delayed), TOTP codes from Google Authenticator or Authy are generated locally and change every 30 seconds.

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

## Common Pitfalls

1. **Not allowing clock drift**: Server and client clocks may differ by a few seconds. If you only accept the current 30-second window, legitimate users get rejected. Accept codes from the previous and next window too (`valid_window=1` in pyotp).
2. **Storing the TOTP secret in plain text**: The TOTP secret is equivalent to a permanent password. Encrypt it at rest in the database using Django's `Fernet` or a field-level encryption library.
3. **Not enforcing code reuse prevention**: Without tracking the last used code, an attacker who intercepts a code has 30 seconds (or longer with clock drift) to replay it. Store and check the last successful timestamp.

## Best Practices

1. **Backup codes**: Provide recovery options.
2. **Allow time drift**: Accept ±1 time window.

## Summary

TOTP combines a shared secret with time to generate codes. Codes are valid for 30 seconds. QR codes simplify setup.

## Resources

- [RFC 6238 TOTP](https://datatracker.ietf.org/doc/html/rfc6238) — TOTP RFC

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*