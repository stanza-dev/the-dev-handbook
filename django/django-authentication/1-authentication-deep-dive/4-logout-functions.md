---
source_course: "django-authentication"
source_lesson: "django-authentication-login-logout-functions"
---

# Login and Logout Functions

## Introduction

Django provides functions to manage user authentication state. Understanding these is key for building login systems.

## Key Concepts

**login()**: Creates authenticated session.

**logout()**: Destroys session.

**authenticate()**: Verifies credentials.

## Real World Context

When building an e-commerce checkout, users who log in mid-session should keep their cart. Django's `login()` function preserves anonymous session data while rotating the session key, so items added before login are not lost -- a detail that matters for conversion rates.

## Deep Dive

### The Login Process

```python
from django.contrib.auth import authenticate, login, logout

def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        
        # Step 1: Verify credentials
        user = authenticate(request, username=username, password=password)
        
        if user is not None:
            # Step 2: Create session
            login(request, user)
            return redirect('dashboard')
        else:
            messages.error(request, 'Invalid credentials')
    
    return render(request, 'login.html')
```

### The Logout Process

```python
def logout_view(request):
    logout(request)  # Clears session
    messages.success(request, 'You have been logged out.')
    return redirect('home')
```

### What login() Does

```python
# login() does the following:
# 1. Saves user ID to session
# 2. Rotates session key (security)
# 3. Sets _auth_user_backend in session
# 4. Sets _auth_user_hash (invalidates on password change)
```

## Common Pitfalls

1. **Calling `login()` without `authenticate()` first**: Skipping `authenticate()` bypasses backend verification and the `user_logged_in` signal, which breaks audit logging and third-party integrations.
2. **Allowing logout via GET requests**: A malicious page can embed `<img src="/logout/">` and log users out. Always require POST for logout and verify the CSRF token.
3. **Not passing `request` to `authenticate()`**: Some backends (like django-axes) use the request for rate limiting. Omitting it silently disables those protections.

## Best Practices

1. **Always use authenticate() first**: Don't call login() with unverified users.
2. **Pass request to authenticate()**: For backend-specific handling.
3. **Use @require_POST for logout**: Prevent CSRF via GET.

## Summary

Use authenticate() to verify credentials, then login() to create the session. logout() clears the session and rotates the key for security.

## Resources

- [Auth Functions](https://docs.djangoproject.com/en/6.0/topics/auth/default/#how-to-log-a-user-in) — Django auth functions

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*