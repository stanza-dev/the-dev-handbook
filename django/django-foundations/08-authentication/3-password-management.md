---
source_course: "django-foundations"
source_lesson: "django-foundations-password-management"
---

# Password Management

## Introduction

Django provides a complete password management system that handles hashing, validation, and reset flows. Understanding how Django stores and validates passwords is essential for building secure applications.

## Key Concepts

- **Password Hashing**: Converting a plain-text password into an irreversible hash for secure storage.
- **Password Validators**: Rules that enforce password strength (length, complexity, common patterns).
- **Password Reset Flow**: A multi-step process using email tokens to let users reset forgotten passwords.

## Real World Context

Password security is one of the most critical aspects of any web application. Data breaches often expose password databases, and proper hashing ensures that even if the database is compromised, passwords remain protected. Django handles this correctly by default.

## Deep Dive

### How Django Stores Passwords

Django never stores plain-text passwords. It uses PBKDF2 by default:

```python
# What's stored in the database:
# algorithm$iterations$salt$hash
# pbkdf2_sha256$870000$salt123$hashvalue...

# Creating users with hashed passwords
from django.contrib.auth.models import User

# CORRECT: Password is hashed automatically
user = User.objects.create_user(
    username='alice',
    password='secure_password_123'
)

# WRONG: Password stored as plain text!
# user = User(username='alice', password='secure_password_123')
# user.save()  # This does NOT hash the password!
```

### Password Hashers

Configure hashers in order of preference:

```python
# settings.py
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',      # Default
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.Argon2PasswordHasher',       # Requires argon2-cffi
    'django.contrib.auth.hashers.BCryptSHA256Hasher',         # Requires bcrypt
    'django.contrib.auth.hashers.ScryptPasswordHasher',
]
```

To use Argon2 (recommended for new projects):

```bash
pip install argon2-cffi
```

```python
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
]
```

### Password Validators

Django 6.0 includes four built-in validators:

```python
# settings.py
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
        # Rejects passwords similar to username/email
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {'min_length': 10},  # Default is 8
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
        # Checks against 20,000 common passwords
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
        # Rejects all-numeric passwords
    },
]
```

### Changing Passwords Programmatically

Use `set_password()` to change a user's password and `check_password()` to verify one. Both methods handle hashing automatically.

```python
user = User.objects.get(username='alice')

# Change password (hashes automatically)
user.set_password('new_secure_password')
user.save()

# Check a password
user.check_password('new_secure_password')  # True
user.check_password('wrong_password')        # False
```

### Password Reset Flow

Django provides a complete password reset flow via email:

```python
# urls.py
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('password_reset/',
         auth_views.PasswordResetView.as_view(),
         name='password_reset'),
    path('password_reset/done/',
         auth_views.PasswordResetDoneView.as_view(),
         name='password_reset_done'),
    path('reset/<uidb64>/<token>/',
         auth_views.PasswordResetConfirmView.as_view(),
         name='password_reset_confirm'),
    path('reset/done/',
         auth_views.PasswordResetCompleteView.as_view(),
         name='password_reset_complete'),
]
```

Configure email:

```python
# settings.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.example.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'noreply@example.com'
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_PASSWORD')

# For development, print emails to console:
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

### Password Change View

Django provides a built-in view for logged-in users to change their password by entering their current password first.

```python
# urls.py
urlpatterns = [
    path('password_change/',
         auth_views.PasswordChangeView.as_view(),
         name='password_change'),
    path('password_change/done/',
         auth_views.PasswordChangeDoneView.as_view(),
         name='password_change_done'),
]
```

The password change view requires the user to be logged in and enter their current password.

## Common Pitfalls

- **Using `User(password='...')` instead of `create_user()`**: The `User()` constructor does not hash the password. Always use `User.objects.create_user()` or `user.set_password()`.
- **Removing all password validators**: Validators protect against weak passwords. Keep at least `MinimumLengthValidator` and `CommonPasswordValidator` enabled.
- **Not configuring email for password reset**: The password reset flow sends an email with a token. Without email configuration, the reset silently fails.

## Best Practices

- **Use Argon2 as the primary hasher**: Install `argon2-cffi` and put `Argon2PasswordHasher` first in `PASSWORD_HASHERS` for stronger protection.
- **Set a minimum password length of 10+**: The default of 8 characters is considered insufficient for modern security standards.
- **Use the console email backend during development**: Set `EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'` to see password reset emails in your terminal.

## Summary

- Django **hashes passwords automatically** using PBKDF2 by default, never storing plain text
- Always use `create_user()` or `set_password()` to ensure passwords are hashed
- Configure `AUTH_PASSWORD_VALIDATORS` to enforce minimum length, complexity, and common password checks
- Django provides a complete **password reset flow** with email-based token verification
- Consider upgrading to **Argon2** for the strongest password hashing available

## Code Examples

**Creating users with hashed passwords and verifying credentials programmatically**

```python
from django.contrib.auth.models import User

# Create a user with hashed password
user = User.objects.create_user(
    username='alice',
    email='alice@example.com',
    password='secure_password_123'
)

# Change password (automatically hashed)
user.set_password('new_password_456')
user.save()

# Verify a password
user.check_password('new_password_456')  # True
```


## Resources

- [Password Management in Django](https://docs.djangoproject.com/en/6.0/topics/auth/passwords/) — Official guide to password hashing, validators, and management

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*