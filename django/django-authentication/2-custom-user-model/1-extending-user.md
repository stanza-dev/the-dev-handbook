---
source_course: "django-authentication"
source_lesson: "django-authentication-extending-user"
---

# Extending the User Model

## Introduction

Django's default User model covers basic authentication, but most real applications need additional fields like profile photos, phone numbers, or email-based login. Choosing the right extension strategy early prevents painful migrations later.

## Key Concepts

**OneToOneField Profile**: Adds extra fields via a separate table linked to User.

**AbstractUser**: Extends the default user while keeping all built-in fields and functionality.

**AbstractBaseUser**: Provides a minimal base for fully custom user models with complete control over fields.

**AUTH_USER_MODEL**: The Django setting that points to your custom user model.

## Real World Context

A healthcare platform needs to store medical license numbers and specialties on each user. Using `AbstractUser` lets the team add these fields while keeping Django's built-in password hashing, permission system, and admin integration -- saving months of custom development.

## Deep Dive

Django provides several ways to customize the User model. Choose based on your needs.

## Option 1: Profile Model (One-to-One)

Best when you can't change the User model but need extra fields:

```python
# models.py
from django.db import models
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver


class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(max_length=500, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)
    phone = models.CharField(max_length=20, blank=True)
    
    def __str__(self):
        return f"{self.user.username}'s profile"


# Auto-create profile when user is created
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

Usage:
```python
user = User.objects.get(username='john')
print(user.profile.bio)
user.profile.phone = '555-1234'
user.profile.save()
```

## Option 2: AbstractUser (Recommended for New Projects)

Extend `AbstractUser` to add fields while keeping all default functionality:

```python
# models.py
from django.contrib.auth.models import AbstractUser
from django.db import models


class User(AbstractUser):
    """Custom user model with additional fields."""
    
    # Add custom fields
    bio = models.TextField(max_length=500, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)
    phone = models.CharField(max_length=20, blank=True)
    
    # Modify existing field
    email = models.EmailField(unique=True)  # Make email unique
    
    class Meta:
        db_table = 'users'  # Custom table name
```

```python
# settings.py
AUTH_USER_MODEL = 'myapp.User'
```

**Important**: Set `AUTH_USER_MODEL` before your first migration!

## Option 3: AbstractBaseUser (Full Control)

Start from scratch with complete control over fields and behavior:

```python
from django.contrib.auth.models import AbstractBaseUser, PermissionsMixin, BaseUserManager
from django.db import models


class UserManager(BaseUserManager):
    """Custom manager for User model."""
    
    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError('Email is required')
        
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user
    
    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        
        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True')
        
        return self.create_user(email, password, **extra_fields)


class User(AbstractBaseUser, PermissionsMixin):
    """Custom user model using email as username."""
    
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=30, blank=True)
    last_name = models.CharField(max_length=30, blank=True)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    date_joined = models.DateTimeField(auto_now_add=True)
    
    objects = UserManager()
    
    USERNAME_FIELD = 'email'  # Use email for authentication
    REQUIRED_FIELDS = []  # Fields required for createsuperuser
    
    class Meta:
        db_table = 'users'
    
    def __str__(self):
        return self.email
    
    def get_full_name(self):
        return f"{self.first_name} {self.last_name}".strip() or self.email
    
    def get_short_name(self):
        return self.first_name or self.email.split('@')[0]
```

## Comparing Approaches

| Approach | When to Use | Pros | Cons |
|----------|-------------|------|------|
| **Profile** | Can't modify User, third-party apps | Easy, non-invasive | Extra JOIN for every query |
| **AbstractUser** | New projects needing extra fields | Simple, keeps all features | Must decide before first migration |
| **AbstractBaseUser** | Need email login, custom fields | Full control | More code, must implement features |

## Referencing the User Model

Always use these methods to reference the User model:

```python
# In models.py - use settings reference
from django.conf import settings

class Article(models.Model):
    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE
    )

# In views/elsewhere - get the model class
from django.contrib.auth import get_user_model

User = get_user_model()
users = User.objects.all()
```

## Common Pitfalls

1. **Starting with the default User and switching later**: Changing `AUTH_USER_MODEL` after the first migration requires resetting every migration that references User. Always create a custom user model at the start of a project, even if it is just `class User(AbstractUser): pass`.
2. **Importing `User` directly instead of using `get_user_model()`**: Hard-coding `from django.contrib.auth.models import User` breaks your code if anyone swaps the user model. Use `get_user_model()` in views and `settings.AUTH_USER_MODEL` in model ForeignKeys.
3. **Adding required fields without defaults to AbstractUser**: New required fields on the user model break `createsuperuser`. Always provide defaults or make new fields nullable/blank.

## Best Practices

1. **Always create a custom user model**: Even if you do not need extra fields yet, `class User(AbstractUser): pass` makes future changes trivial.
2. **Use `AbstractUser` unless you need email-only login**: It keeps all default behavior. Only use `AbstractBaseUser` when you need to remove the username field entirely.
3. **Reference users with `settings.AUTH_USER_MODEL`**: In model ForeignKeys, always use the string reference so your models are portable across projects.

## Summary

- Django offers three approaches to extend the User model: Profile (OneToOne), AbstractUser, and AbstractBaseUser.
- `AbstractUser` is recommended for most new projects because it keeps all defaults while allowing custom fields.
- Always set `AUTH_USER_MODEL` before the first migration.
- Use `get_user_model()` and `settings.AUTH_USER_MODEL` instead of importing User directly.
- For existing projects that cannot change the user model, use a OneToOneField Profile.

## Code Examples

**Custom user model extending AbstractUser with additional profile fields**

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    date_of_birth = models.DateField(null=True, blank=True)

    class Meta:
        db_table = 'auth_user'
```


## Resources

- [Substituting a Custom User Model](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/#substituting-a-custom-user-model) — Official guide to custom user models

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*