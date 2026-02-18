---
source_course: "django-authentication"
source_lesson: "django-authentication-password-reset"
---

# Password Reset Flow

## Introduction

Django provides built-in views and forms for secure password reset via email.

## Key Concepts

**Token-based Reset**: Secure, one-time use tokens.

**Password Reset Views**: Built-in CBVs.

## Deep Dive

### URL Configuration

```python
# urls.py
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('password_reset/', auth_views.PasswordResetView.as_view(),
         name='password_reset'),
    path('password_reset/done/', auth_views.PasswordResetDoneView.as_view(),
         name='password_reset_done'),
    path('reset/<uidb64>/<token>/', auth_views.PasswordResetConfirmView.as_view(),
         name='password_reset_confirm'),
    path('reset/done/', auth_views.PasswordResetCompleteView.as_view(),
         name='password_reset_complete'),
]
```

### Email Template

```html
<!-- templates/registration/password_reset_email.html -->
Hello,

You requested a password reset for {{ user.email }}.

Click here to reset: {{ protocol }}://{{ domain }}{% url 'password_reset_confirm' uidb64=uid token=token %}

This link expires in {{ expiry }} hours.
```

### Token Security

```python
# settings.py
PASSWORD_RESET_TIMEOUT = 3600  # 1 hour (in seconds)
```

## Best Practices

1. **Use HTTPS**: For all reset links.
2. **Limit token lifetime**: Default 3 days is too long.
3. **Rate limit requests**: Prevent email flooding.

## Summary

Use Django's built-in password reset views. Configure short token timeout. Always use HTTPS for reset links.

## Resources

- [Password Reset](https://docs.djangoproject.com/en/6.0/topics/auth/default/#django.contrib.auth.views.PasswordResetView) — Password reset views

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*