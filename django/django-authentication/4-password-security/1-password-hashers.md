---
source_course: "django-authentication"
source_lesson: "django-authentication-password-hashers"
---

# Password Hashers

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
  Stored: "pbkdf2_sha256$600000$salt$hash..."
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
pbkdf2_sha256$600000$abc123xyz$VhLmf8...
```

## Increasing Security (Iterations)

For Django 6.0, PBKDF2 uses 1,200,000 iterations by default. You can increase this:

```python
# hashers.py
from django.contrib.auth.hashers import PBKDF2PasswordHasher


class MyPBKDF2PasswordHasher(PBKDF2PasswordHasher):
    iterations = 1500000  # Increase iterations


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
# 'pbkdf2_sha256$600000$...'

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
# If password was PBKDF2 with 480000 iterations,
# it's re-hashed with current settings (600000 iterations)
```

## Resources

- [Password Management](https://docs.djangoproject.com/en/6.0/topics/auth/passwords/) — Official password management documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*