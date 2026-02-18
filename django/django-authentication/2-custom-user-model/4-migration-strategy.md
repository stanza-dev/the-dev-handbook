---
source_course: "django-authentication"
source_lesson: "django-authentication-migration-strategy"
---

# User Model Migration Strategy

## Introduction

Changing user models mid-project is complex. Understanding migration strategies helps avoid pitfalls.

## Key Concepts

**AUTH_USER_MODEL**: Must be set before first migration.

**Data Migration**: Moving data between user models.

## Deep Dive

### Starting Fresh (Recommended)

```python
# 1. Create custom user model FIRST
# accounts/models.py
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    pass  # Add fields later

# 2. Configure in settings.py
AUTH_USER_MODEL = 'accounts.User'

# 3. Then run first migration
# python manage.py makemigrations
# python manage.py migrate
```

### Migrating Existing Project

```python
# WARNING: Complex process, consider carefully

# Option 1: Database copy (if possible)
# 1. Export existing user data
# 2. Create new custom model
# 3. Reset migrations (dangerous!)
# 4. Import data

# Option 2: Keep using auth.User with Profile
class Profile(models.Model):
    user = models.OneToOneField(
        'auth.User',
        on_delete=models.CASCADE
    )
    # Add all extra fields here
```

### Setting Model Early

```python
# Even if you don't need customization now,
# create a minimal custom user model:

class User(AbstractUser):
    class Meta:
        db_table = 'auth_user'  # Use same table name

# This makes future changes much easier
```

## Best Practices

1. **Always create custom user first**: Before any migrations.
2. **Use Profile for existing projects**: Avoid risky migrations.
3. **Test thoroughly**: User model changes are critical.

## Summary

Create custom user models before first migration. For existing projects, use Profile models. Never rush user model changes.

## Resources

- [Changing AUTH_USER_MODEL](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/#changing-to-a-custom-user-model-mid-project) — Mid-project user model changes

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*