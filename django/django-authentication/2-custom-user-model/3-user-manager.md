---
source_course: "django-authentication"
source_lesson: "django-authentication-user-manager"
---

# Custom User Manager

## Introduction

Custom user managers handle the creation of user instances. They're essential when using AbstractBaseUser.

## Key Concepts

**BaseUserManager**: Base class for user managers.

**normalize_email()**: Lowercases the domain part of email.

## Deep Dive

### Complete Manager Implementation

```python
from django.contrib.auth.models import BaseUserManager

class CustomUserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        """Create and return a regular user."""
        if not email:
            raise ValueError('Email address is required')
        
        email = self.normalize_email(email)
        extra_fields.setdefault('is_active', True)
        
        user = self.model(email=email, **extra_fields)
        user.set_password(password)  # Hashes the password
        user.save(using=self._db)
        return user
    
    def create_superuser(self, email, password=None, **extra_fields):
        """Create and return a superuser."""
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        
        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True.')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True.')
        
        return self.create_user(email, password, **extra_fields)
    
    def get_by_natural_key(self, email):
        """Case-insensitive email lookup."""
        return self.get(email__iexact=email)
```

### Using the Manager

```python
class User(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=255)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    
    objects = CustomUserManager()  # Attach manager
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['name']  # For createsuperuser
```

## Best Practices

1. **Always use set_password()**: Never store plain text passwords.
2. **Use normalize_email()**: Ensures consistent email format.
3. **Validate in manager**: Check required fields.

## Summary

Custom managers handle user creation. Use set_password() for security. Include validation for required fields.

## Resources

- [User Manager](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/#writing-a-manager-for-a-custom-user-model) — Custom user manager documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*