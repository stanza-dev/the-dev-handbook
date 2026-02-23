---
source_course: "django-authentication"
source_lesson: "django-authentication-password-validation"
---

# Password Validation

## Introduction

Strong passwords are the first line of defense against account takeover. Django's password validation framework rejects weak passwords at registration and password change, and it is fully customizable to enforce your organization's security policy.

## Key Concepts

**AUTH_PASSWORD_VALIDATORS**: Setting that lists active validators and their options.

**UserAttributeSimilarityValidator**: Rejects passwords too similar to the username, email, or name.

**MinimumLengthValidator**: Enforces a minimum password length (default 8 characters).

**CommonPasswordValidator**: Rejects the 20,000 most common passwords.

**Custom Validator**: Any class with `validate()` and `get_help_text()` methods.

## Real World Context

A fintech application requires passwords with at least 12 characters, one uppercase letter, one digit, and one special character. By writing three small custom validators and adding them to `AUTH_PASSWORD_VALIDATORS`, the policy is enforced consistently across signup, password change, password reset, and the Django admin.

## Deep Dive

Django includes validators to enforce password policies. Configure them to meet your security requirements.

## Default Validators

```python
# settings.py
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 8,
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

## Built-in Validators

| Validator | Description |
|-----------|-------------|
| `UserAttributeSimilarityValidator` | Password can't be similar to user info |
| `MinimumLengthValidator` | Enforces minimum length |
| `CommonPasswordValidator` | Rejects 20,000 common passwords |
| `NumericPasswordValidator` | Password can't be entirely numeric |

## Customizing Validators

```python
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
            'min_length': 12,  # Stronger requirement
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
        'OPTIONS': {
            'password_list_path': '/path/to/custom/passwords.txt',
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

## Custom Password Validator

```python
# validators.py
from django.core.exceptions import ValidationError
import re


class UppercaseValidator:
    """Require at least one uppercase letter."""
    
    def validate(self, password, user=None):
        if not re.search(r'[A-Z]', password):
            raise ValidationError(
                'Password must contain at least one uppercase letter.',
                code='password_no_uppercase',
            )
    
    def get_help_text(self):
        return 'Your password must contain at least one uppercase letter.'


class LowercaseValidator:
    """Require at least one lowercase letter."""
    
    def validate(self, password, user=None):
        if not re.search(r'[a-z]', password):
            raise ValidationError(
                'Password must contain at least one lowercase letter.',
                code='password_no_lowercase',
            )
    
    def get_help_text(self):
        return 'Your password must contain at least one lowercase letter.'


class SpecialCharacterValidator:
    """Require at least one special character."""
    
    def __init__(self, special_characters='!@#$%^&*()_+-=[]{}|;:,.<>?'):
        self.special_characters = special_characters
    
    def validate(self, password, user=None):
        if not any(char in self.special_characters for char in password):
            raise ValidationError(
                'Password must contain at least one special character.',
                code='password_no_special',
            )
    
    def get_help_text(self):
        return f'Your password must contain at least one special character: {self.special_characters}'


class NoRepeatingCharactersValidator:
    """Prevent repeated characters."""
    
    def __init__(self, max_repeats=3):
        self.max_repeats = max_repeats
    
    def validate(self, password, user=None):
        for i in range(len(password) - self.max_repeats + 1):
            if len(set(password[i:i+self.max_repeats])) == 1:
                raise ValidationError(
                    f'Password cannot contain {self.max_repeats} or more repeating characters.',
                    code='password_repeating',
                )
    
    def get_help_text(self):
        return f'Your password cannot have {self.max_repeats} or more repeating characters.'
```

```python
# settings.py
AUTH_PASSWORD_VALIDATORS = [
    # ... default validators ...
    {
        'NAME': 'myapp.validators.UppercaseValidator',
    },
    {
        'NAME': 'myapp.validators.LowercaseValidator',
    },
    {
        'NAME': 'myapp.validators.SpecialCharacterValidator',
        'OPTIONS': {
            'special_characters': '!@#$%^&*()',
        }
    },
]
```

## Using Validators Programmatically

```python
from django.contrib.auth.password_validation import (
    validate_password,
    get_password_validators,
    password_validators_help_texts
)
from django.core.exceptions import ValidationError

# Validate a password
try:
    validate_password('weakpass', user=user)
except ValidationError as e:
    print(e.messages)
    # ['This password is too short...',
    #  'This password is too common.']

# Get help text for all validators
help_texts = password_validators_help_texts()
# ['Your password must contain at least 8 characters.',
#  'Your password can't be a commonly used password.', ...]
```

## Common Pitfalls

1. **Disabling validators in development**: If you remove validators locally, you create test accounts with weak passwords like `123`. Those same passwords might end up in staging or production fixtures.
2. **Not calling `validate_password()` in custom forms**: Django's built-in auth forms call validators automatically, but if you build a custom registration form and skip `validate_password()`, the validators are bypassed entirely.
3. **Over-restricting passwords**: Requiring uppercase, lowercase, digits, and special characters frustrates users and pushes them toward predictable patterns like `Password1!`. Length is more important than complexity.

## Best Practices

1. **Prioritize length over complexity**: A 16-character passphrase is stronger than an 8-character complex password. Set `MinimumLengthValidator` to 10 or 12.
2. **Use all four default validators**: They complement each other -- similarity, length, common list, and numeric checks together catch most weak passwords.
3. **Show help text to users**: Call `password_validators_help_texts()` and display the rules on the registration form so users know the requirements upfront.

## Summary

- Django validates passwords using validators listed in `AUTH_PASSWORD_VALIDATORS`.
- Four built-in validators cover similarity, length, common passwords, and numeric-only checks.
- Custom validators need only `validate()` and `get_help_text()` methods.
- Always call `validate_password()` in custom forms to enforce the policy.
- Prefer longer minimum length over complex character requirements.

## Resources

- [Password Validation](https://docs.djangoproject.com/en/6.0/topics/auth/passwords/#password-validation) — Official password validation documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*