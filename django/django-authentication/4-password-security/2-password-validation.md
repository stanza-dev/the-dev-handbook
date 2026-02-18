---
source_course: "django-authentication"
source_lesson: "django-authentication-password-validation"
---

# Password Validation

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

## Resources

- [Password Validation](https://docs.djangoproject.com/en/6.0/topics/auth/passwords/#password-validation) — Official password validation documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*