---
source_course: "django-authentication"
source_lesson: "django-authentication-social-auth-intro"
---

# Social Authentication with django-allauth

## Introduction

Social authentication lets users log in with their existing Google, GitHub, or Facebook accounts instead of creating yet another username and password. django-allauth is the most popular package for adding social login to Django, supporting over 80 providers.

## Key Concepts

**django-allauth**: A comprehensive Django package for authentication, registration, and social login.

**SocialApp**: Model storing provider credentials (client ID and secret).

**Provider**: A third-party service (Google, GitHub, etc.) that authenticates users.

**Callback URL**: The URL the provider redirects to after authentication.

**SOCIALACCOUNT_PROVIDERS**: Setting to configure scopes, parameters, and credentials for each provider.

## Real World Context

A developer tools startup adding "Login with GitHub" saw a 35% increase in signups because developers prefer not to create new accounts. django-allauth handles the OAuth flow, user creation, and account linking, so the team shipped the feature in a day instead of building OAuth from scratch.

## Deep Dive

Allow users to log in with their existing accounts from Google, GitHub, Facebook, and many other providers.

## Installing django-allauth

```bash
pip install django-allauth
```

## Basic Configuration

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.sites',
    # ...
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    # Providers
    'allauth.socialaccount.providers.google',
    'allauth.socialaccount.providers.github',
    'allauth.socialaccount.providers.facebook',
]

MIDDLEWARE = [
    # ...
    'allauth.account.middleware.AccountMiddleware',
]

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
]

SITE_ID = 1

# Allauth settings
ACCOUNT_LOGIN_ON_EMAIL_CONFIRMATION = True
ACCOUNT_LOGOUT_ON_GET = True
ACCOUNT_UNIQUE_EMAIL = True
ACCOUNT_EMAIL_REQUIRED = True
ACCOUNT_USERNAME_REQUIRED = False
ACCOUNT_AUTHENTICATION_METHOD = 'email'
ACCOUNT_EMAIL_VERIFICATION = 'mandatory'

# Redirect URLs
LOGIN_REDIRECT_URL = '/dashboard/'
ACCOUNT_LOGOUT_REDIRECT_URL = '/'
```

## URL Configuration

```python
# urls.py
from django.urls import path, include

urlpatterns = [
    # ...
    path('accounts/', include('allauth.urls')),
]
```

## Setting Up Google OAuth

1. Go to Google Cloud Console
2. Create a new project
3. Enable Google+ API
4. Create OAuth 2.0 credentials
5. Set redirect URI: `http://localhost:8000/accounts/google/login/callback/`

```python
# settings.py
SOCIALACCOUNT_PROVIDERS = {
    'google': {
        'SCOPE': [
            'profile',
            'email',
        ],
        'AUTH_PARAMS': {
            'access_type': 'online',
        },
        'OAUTH_PKCE_ENABLED': True,
    }
}
```

Add credentials in Django admin:
1. Go to `/admin/socialaccount/socialapp/`
2. Add a new Social Application
3. Select Google as provider
4. Enter Client ID and Secret Key
5. Associate with your Site

## Setting Up GitHub OAuth

1. Go to GitHub Developer Settings
2. Create a new OAuth App
3. Set callback URL: `http://localhost:8000/accounts/github/login/callback/`

```python
# settings.py
SOCIALACCOUNT_PROVIDERS = {
    'github': {
        'SCOPE': [
            'user',
            'read:user',
            'user:email',
        ],
    },
}
```

## Login Templates

```html
<!-- templates/account/login.html -->
{% extends 'base.html' %}
{% load socialaccount %}

{% block content %}
<h1>Login</h1>

<!-- Standard login form -->
<form method="post" action="{% url 'account_login' %}">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>

<hr>
<h2>Or login with:</h2>

<!-- Social login buttons -->
{% get_providers as socialaccount_providers %}
{% for provider in socialaccount_providers %}
    <a href="{% provider_login_url provider.id %}" class="btn btn-{{ provider.id }}">
        Login with {{ provider.name }}
    </a>
{% endfor %}

<!-- Or direct links -->
<a href="{% provider_login_url 'google' %}">Login with Google</a>
<a href="{% provider_login_url 'github' %}">Login with GitHub</a>
{% endblock %}
```

## Handling Social Account Data

```python
# signals.py
from allauth.account.signals import user_signed_up
from allauth.socialaccount.signals import social_account_added
from django.dispatch import receiver


@receiver(user_signed_up)
def handle_user_signed_up(request, user, **kwargs):
    """Handle new user registration."""
    # Create profile, send welcome email, etc.
    Profile.objects.create(user=user)


@receiver(social_account_added)
def handle_social_account_added(request, sociallogin, **kwargs):
    """Handle when user connects a social account."""
    user = sociallogin.user
    social_data = sociallogin.account.extra_data
    
    # Update profile with social data
    if sociallogin.account.provider == 'google':
        user.profile.avatar_url = social_data.get('picture')
        user.profile.save()
```

## Custom Adapter

```python
# adapters.py
from allauth.socialaccount.adapter import DefaultSocialAccountAdapter


class CustomSocialAccountAdapter(DefaultSocialAccountAdapter):
    def pre_social_login(self, request, sociallogin):
        """Called before social login completes."""
        # Connect social account to existing user with same email
        email = sociallogin.account.extra_data.get('email')
        if email:
            try:
                user = User.objects.get(email=email)
                sociallogin.connect(request, user)
            except User.DoesNotExist:
                pass
    
    def populate_user(self, request, sociallogin, data):
        """Populate user from social account data."""
        user = super().populate_user(request, sociallogin, data)
        user.first_name = data.get('first_name', '')
        user.last_name = data.get('last_name', '')
        return user
```

```python
# settings.py
SOCIALACCOUNT_ADAPTER = 'myapp.adapters.CustomSocialAccountAdapter'
```

## Common Pitfalls

1. **Forgetting `AccountMiddleware`**: django-allauth requires `allauth.account.middleware.AccountMiddleware` in `MIDDLEWARE`. Without it, allauth views raise errors that are hard to debug.
2. **Not handling email conflicts**: When a user signs up with email first, then tries "Login with Google" using the same email, the accounts must be linked. Without `SOCIALACCOUNT_ADAPTER` customization, users get a confusing error.
3. **Skipping `ACCOUNT_EMAIL_VERIFICATION`**: Without email verification, an attacker can create an account with someone else's email, then use social login to take over the original account.

## Best Practices

1. **Set `ACCOUNT_EMAIL_REQUIRED = True` and `ACCOUNT_EMAIL_VERIFICATION = 'mandatory'`**: This ensures email addresses are verified, preventing account takeover via email spoofing.
2. **Store credentials in environment variables**: Use `SOCIALACCOUNT_PROVIDERS` with `APP.client_id` and `APP.secret` referencing `os.environ` instead of storing secrets in the database.
3. **Write a custom `SocialAccountAdapter`**: Override `pre_social_login()` to automatically link social accounts to existing users with the same verified email address.

## Summary

- django-allauth provides social login for 80+ providers with minimal configuration.
- Install allauth, add it to `INSTALLED_APPS`, configure `AUTHENTICATION_BACKENDS`, and include URLs.
- Configure each provider's credentials via `SOCIALACCOUNT_PROVIDERS` in settings.
- Handle email conflicts with a custom `SocialAccountAdapter`.
- Always verify emails and store credentials in environment variables.

## Code Examples

**Setting up django-allauth for social authentication with Google and GitHub**

```python
# settings.py
INSTALLED_APPS = [
    ...
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
    'allauth.socialaccount.providers.github',
]

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
]
```


## Resources

- [django-allauth](https://django-allauth.readthedocs.io/) — Official django-allauth documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*