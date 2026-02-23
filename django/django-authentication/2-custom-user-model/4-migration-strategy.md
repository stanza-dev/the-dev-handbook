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

## Real World Context

A startup that began with Django's default `auth.User` now needs to add a `company` field and use email-based login. Changing `AUTH_USER_MODEL` mid-project requires resetting every migration that references the user model -- a process that often takes days in production. The Profile model pattern avoids this pain entirely.

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

## Common Pitfalls

1. **Changing `AUTH_USER_MODEL` after the first migration**: Django does not support swapping the user model after migrations have been applied. All ForeignKeys to the old model break, and data must be manually migrated.
2. **Hardcoding `from django.contrib.auth.models import User`**: If you later switch to a custom model, every import breaks. Always use `get_user_model()` in code and `settings.AUTH_USER_MODEL` in models.
3. **Forgetting to create the custom model before `migrate`**: Running `migrate` without the custom model registered in `AUTH_USER_MODEL` creates the default `auth_user` table, making the switch much harder.

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