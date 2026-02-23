---
source_course: "django-authentication"
source_lesson: "django-authentication-password-hashers"
---

# Password Hashers

## Introduction

Passwords must never be stored in plain text. Django's password hashing system automatically applies industry-standard algorithms like PBKDF2, Argon2, and bcrypt, so even if your database is compromised, user passwords remain protected.

## Key Concepts

**Password Hasher**: An algorithm that converts a plain-text password into an irreversible hash.

**PBKDF2**: Django's default hasher, using SHA-256 with 1,200,000 iterations.

**Argon2**: The winner of the Password Hashing Competition, recommended for new projects.

**Salt**: Random data added to the password before hashing to prevent rainbow table attacks.

**PASSWORD_HASHERS**: Setting that controls which hashers are available and their priority order.

## Real World Context

When a major company suffers a data breach, attackers try to crack the stolen password hashes. With PBKDF2 at 1.2 million iterations, cracking a single strong password takes weeks on consumer hardware. Switching to Argon2 makes it even harder because the algorithm is memory-hard, defeating GPU-based attacks.

## Deep Dive

Django uses secure password hashing to protect user credentials. Understanding hashers helps you configure security appropriately.

## How Password Hashing Works

```
User enters: "mypassword123"
                │
                ▼
        ┌───────────────┐
        │   Hasher      │
        │ (PBKDF2+SHA256)│
        └───────────────┘
                │
                ▼
  Stored: "pbkdf2_sha256$1200000$salt$hash..."
        │         │        │     │
     algorithm iterations salt  hash
```

## Default Hashers

```python
# settings.py (default configuration)
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',  # Default
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.ScryptPasswordHasher',
]
```

The first hasher is used for new passwords. Others are for verifying existing passwords (during migration).

## Recommended: Argon2

Argon2 is the most secure option:

```bash
pip install argon2-cffi
```

```python
# settings.py
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
]
```

## Password Storage Format

```
<algorithm>$<iterations>$<salt>$<hash>

Example:
pbkdf2_sha256$1200000$abc123xyz$VhLmf8...
```

## Increasing Security (Iterations)

For Django 6.0, PBKDF2 uses 1,200,000 iterations by default. You can increase this:

```python
# hashers.py
from django.contrib.auth.hashers import PBKDF2PasswordHasher


class MyPBKDF2PasswordHasher(PBKDF2PasswordHasher):
    iterations = 2400000  # Increase iterations


# settings.py
PASSWORD_HASHERS = [
    'myapp.hashers.MyPBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',  # For existing passwords
]
```

## Password Hashing Functions

```python
from django.contrib.auth.hashers import (
    make_password,
    check_password,
    is_password_usable
)

# Hash a password
hashed = make_password('mypassword')
# 'pbkdf2_sha256$1200000$...'

# Verify a password
check_password('mypassword', hashed)  # True
check_password('wrongpassword', hashed)  # False

# Check if password is usable (not marked as unusable)
is_password_usable(hashed)  # True

# Mark password as unusable (prevent login)
unusable = make_password(None)
is_password_usable(unusable)  # False
```

## User Model Methods

```python
# Set password (hashes automatically)
user.set_password('newpassword')
user.save()

# Check password
user.check_password('password')  # True/False

# Set unusable password (e.g., for social auth users)
user.set_unusable_password()
user.save()

# Check if password is usable
user.has_usable_password()  # True/False
```

## Password Upgrade

Django automatically upgrades password hashes when:
1. User logs in successfully
2. A newer/stronger hasher is first in `PASSWORD_HASHERS`

```python
# This happens automatically during authentication
user = authenticate(request, username='john', password='secret')
# If password was PBKDF2 with 870000 iterations,
# it's re-hashed with current settings (1200000 iterations)
```

## Common Pitfalls

1. **Using `make_password()` with an explicit salt**: Django generates a cryptographically random salt automatically. Providing your own salt (especially a static one) defeats the purpose and makes hashes predictable.
2. **Removing old hashers from `PASSWORD_HASHERS`**: The list must include all algorithms that existing passwords use. If you remove PBKDF2 after switching to Argon2, existing users cannot log in until they reset their password.
3. **Comparing hashes directly**: Never use `==` to compare password hashes. Use `check_password()` which handles salt extraction, algorithm detection, and timing-safe comparison.

## Best Practices

1. **Put Argon2 first in `PASSWORD_HASHERS`**: New passwords use the first hasher. Existing passwords are automatically upgraded on the next successful login.
2. **Keep old hashers in the list**: This ensures backward compatibility while gradually migrating all users to the strongest algorithm.
3. **Increase iterations periodically**: As hardware gets faster, bump the iteration count. Django increases the default with each release.

## Summary

- Django hashes passwords automatically using the first algorithm in `PASSWORD_HASHERS`.
- Argon2 is the recommended hasher for new projects; install it with `pip install argon2-cffi`.
- Password hashes are stored in the format `algorithm$iterations$salt$hash`.
- Django upgrades hashes transparently when users log in and a stronger hasher is available.
- Never store, compare, or manipulate password hashes directly -- use `set_password()` and `check_password()`.

## Code Examples

**Configuring password hashers and validators in Django 6 settings**

```python
# settings.py
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.ScryptPasswordHasher',
]

AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
     'OPTIONS': {'min_length': 10}},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]
```


## Resources

- [Password Management](https://docs.djangoproject.com/en/6.0/topics/auth/passwords/) — Official password management documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*