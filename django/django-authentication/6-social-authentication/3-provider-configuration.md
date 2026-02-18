---
source_course: "django-authentication"
source_lesson: "django-authentication-provider-configuration"
---

# Provider Configuration

## Introduction

Configure social providers correctly for secure authentication.

## Key Concepts

**SocialApp**: Stores provider credentials.

**Client ID/Secret**: OAuth credentials.

## Deep Dive

### Via Django Admin

```
1. Go to /admin/socialaccount/socialapp/
2. Add new Social Application
3. Select provider (Google, GitHub, etc.)
4. Enter Client ID and Secret
5. Select Sites
```

### Via Settings

```python
# settings.py
SOCIALACCOUNT_PROVIDERS = {
    'google': {
        'APP': {
            'client_id': os.environ['GOOGLE_CLIENT_ID'],
            'secret': os.environ['GOOGLE_CLIENT_SECRET'],
        },
        'SCOPE': ['profile', 'email'],
        'AUTH_PARAMS': {'access_type': 'online'},
    },
    'github': {
        'APP': {
            'client_id': os.environ['GITHUB_CLIENT_ID'],
            'secret': os.environ['GITHUB_CLIENT_SECRET'],
        },
        'SCOPE': ['user:email'],
    },
}
```

### Environment Variables

```bash
# .env
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx
GITHUB_CLIENT_ID=xxx
GITHUB_CLIENT_SECRET=xxx
```

## Best Practices

1. **Never commit secrets**: Use environment variables.
2. **Separate dev/prod apps**: Different callback URLs.
3. **Minimal scopes**: Only request what you need.

## Summary

Configure providers via settings (preferred) or admin. Always use environment variables for secrets.

## Resources

- [Provider Configuration](https://django-allauth.readthedocs.io/en/latest/socialaccount/configuration.html) — Allauth provider configuration

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*