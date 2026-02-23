---
source_course: "django-security"
source_lesson: "django-security-password-security"
---

# Password Security

## Introduction

Password security is a critical layer of authentication defense. Django includes a robust password hashing system with pluggable hashers and configurable validators, making it straightforward to enforce strong password policies out of the box.

## Key Concepts

- **Password Hasher**: Algorithm that converts passwords into irreversible hashes for storage (Argon2, PBKDF2, BCrypt).
- **Password Validators**: Configurable rules that enforce complexity, length, and uniqueness requirements.
- **Argon2**: Winner of the Password Hashing Competition; memory-hard algorithm that resists GPU and ASIC attacks.
- **Brute Force Protection**: Rate limiting and lockout mechanisms that prevent automated password guessing.

## Real World Context

Weak password hashing has led to massive credential breaches. When LinkedIn's SHA1-hashed passwords were leaked, millions were cracked within hours. Using a modern memory-hard hasher like Argon2 makes each cracking attempt orders of magnitude more expensive, buying time for users to change compromised passwords.

## Deep Dive

### Password Hashers

```python
# settings.py
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',  # Recommended
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
]

# pip install argon2-cffi  # For Argon2
```

### Password Validators

```python
# settings.py
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
        'OPTIONS': {
            'user_attributes': ('username', 'email', 'first_name', 'last_name'),
            'max_similarity': 0.7,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 12,  # Increase from default 8
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

### Custom Password Validator

```python
# validators.py
from django.core.exceptions import ValidationError
import re


class ComplexityValidator:
    """Require uppercase, lowercase, digit, and special character."""
    
    def validate(self, password, user=None):
        if not re.search(r'[A-Z]', password):
            raise ValidationError(
                'Password must contain at least one uppercase letter.',
                code='no_upper'
            )
        if not re.search(r'[a-z]', password):
            raise ValidationError(
                'Password must contain at least one lowercase letter.',
                code='no_lower'
            )
        if not re.search(r'\d', password):
            raise ValidationError(
                'Password must contain at least one digit.',
                code='no_digit'
            )
        if not re.search(r'[!@#$%^&*(),.?\":{}|<>]', password):
            raise ValidationError(
                'Password must contain at least one special character.',
                code='no_special'
            )
    
    def get_help_text(self):
        return 'Password must contain uppercase, lowercase, digit, and special character.'
```

### Brute Force Protection

```python
# pip install django-axes

# settings.py
INSTALLED_APPS = [
    # ...
    'axes',
]

AUTHENTICATION_BACKENDS = [
    'axes.backends.AxesStandaloneBackend',
    'django.contrib.auth.backends.ModelBackend',
]

# Lock out after 5 failed attempts
AXES_FAILURE_LIMIT = 5

# Lock out for 15 minutes
AXES_COOLOFF_TIME = 0.25  # hours (15 minutes)

# Lock by IP and username combination
AXES_LOCKOUT_PARAMETERS = ['ip_address', 'username']

# Only lock out on POST to login URL
AXES_ONLY_USER_FAILURES = False
```

### Secure Password Reset

```python
# settings.py
PASSWORD_RESET_TIMEOUT = 3600  # Token valid for 1 hour (default: 3 days)

# Use HTTPS for password reset links
if not DEBUG:
    USE_X_FORWARDED_HOST = True
    SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

### Session Security

```python
# settings.py

# Use database sessions for security
SESSION_ENGINE = 'django.contrib.sessions.backends.db'

# Secure cookie settings
SESSION_COOKIE_SECURE = True      # HTTPS only
SESSION_COOKIE_HTTPONLY = True    # No JavaScript access
SESSION_COOKIE_SAMESITE = 'Lax'   # CSRF protection
SESSION_COOKIE_AGE = 1209600      # 2 weeks

# Expire session on browser close
SESSION_EXPIRE_AT_BROWSER_CLOSE = True

# Django's login() automatically rotates the session ID
def login_view(request):
    # ... authenticate user ...
    login(request, user)  # cycle_key() is called internally
```

### Two-Factor Authentication

```python
# pip install django-two-factor-auth

# settings.py
INSTALLED_APPS = [
    # ...
    'django_otp',
    'django_otp.plugins.otp_static',
    'django_otp.plugins.otp_totp',
    'two_factor',
]

LOGIN_URL = 'two_factor:login'
```

## Common Pitfalls

1. **Using the default PBKDF2 when Argon2 is available** — PBKDF2 is safe but Argon2 is significantly harder to crack with GPUs; install argon2-cffi and make it your first hasher.
2. **Setting PASSWORD_RESET_TIMEOUT too high** — The default 3 days gives attackers a large window; reduce to 1 hour for production.
3. **Not validating passwords on the backend** — Client-side validation is easily bypassed; always enforce validators server-side.

## Best Practices

1. **Make Argon2 the primary hasher** — Install argon2-cffi and list it first in PASSWORD_HASHERS.
2. **Increase minimum password length to 12+** — Length is more important than complexity rules for entropy.
3. **Combine hashers with rate limiting** — Use django-axes alongside strong hashing for defense in depth.

## Summary

- Configure Argon2 as your primary hasher and enforce a minimum 12-character password length.
- Use Django's built-in validators and add custom validators for domain-specific requirements.
- Pair strong hashing with brute force protection (django-axes) and two-factor authentication for complete password security.

## Code Examples

**Recommended password hasher and validator configuration with Argon2 as the primary hasher and 12-character minimum**

```python
# settings.py - Secure password configuration
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
]

AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
     'OPTIONS': {'min_length': 12}},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

# pip install argon2-cffi
```


## Resources

- [Password Management](https://docs.djangoproject.com/en/6.0/topics/auth/passwords/) — Django password management

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*