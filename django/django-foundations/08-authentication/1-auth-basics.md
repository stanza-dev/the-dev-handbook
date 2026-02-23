---
source_course: "django-foundations"
source_lesson: "django-foundations-auth-basics"
---

# Django Authentication System

Django includes a robust authentication system that handles user accounts, groups, permissions, and sessions out of the box.

## What's Included

Django's `auth` app provides:
- **User model**: Stores usernames, passwords, emails
- **Password hashing**: Secure password storage
- **Permissions**: Fine-grained access control
- **Groups**: Organize users with shared permissions
- **Views**: Login, logout, password reset
- **Decorators**: Restrict view access

## Configuration

The auth app is included by default:

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',         # Authentication framework
    'django.contrib.contenttypes', # Required by auth
    'django.contrib.sessions',     # Session framework
    ...
]

MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    ...
]
```

## The User Model

Django provides a built-in User model:

```python
from django.contrib.auth.models import User

# Create a user
user = User.objects.create_user(
    username='john',
    email='john@example.com',
    password='secret123'  # Automatically hashed
)

# Create a superuser
from django.contrib.auth.models import User
User.objects.create_superuser(
    username='admin',
    email='admin@example.com',
    password='adminpass'
)

# User attributes
user.username
user.email
user.first_name
user.last_name
user.is_active      # Can login?
user.is_staff       # Can access admin?
user.is_superuser   # Has all permissions?
user.date_joined
user.last_login
```

## Authentication in Views

Check if a user is logged in:

```python
def my_view(request):
    if request.user.is_authenticated:
        # User is logged in
        username = request.user.username
        return HttpResponse(f"Hello, {username}!")
    else:
        # User is anonymous
        return HttpResponse("Please log in.")
```

## Login/Logout

Django provides authentication functions:

```python
from django.contrib.auth import authenticate, login, logout
from django.views.decorators.http import require_POST

def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        
        # Check credentials
        user = authenticate(request, username=username, password=password)
        
        if user is not None:
            login(request, user)  # Create session
            return redirect('home')
        else:
            return render(request, 'login.html', {'error': 'Invalid credentials'})
    
    return render(request, 'login.html')


@require_POST
def logout_view(request):
    logout(request)  # Clear session
    return redirect('home')
```

## Using Built-in Views

Django provides ready-to-use authentication views:

```python
# mysite/urls.py
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
    path('password_change/', auth_views.PasswordChangeView.as_view(), name='password_change'),
    path('password_reset/', auth_views.PasswordResetView.as_view(), name='password_reset'),
]
```

Or include all auth URLs:

```python
urlpatterns = [
    path('accounts/', include('django.contrib.auth.urls')),
]
```

This provides:
- `accounts/login/`
- `accounts/logout/`
- `accounts/password_change/`
- `accounts/password_reset/`

## Required Templates

Create templates for the built-in views:

```html
<!-- templates/registration/login.html -->
<h1>Login</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>
```

## Settings

Django uses several settings to control authentication behavior, including where users are redirected after login and logout.

```python
# settings.py

# Where to redirect after login
LOGIN_REDIRECT_URL = 'home'

# Where to redirect after logout
LOGOUT_REDIRECT_URL = 'home'

# Login URL for @login_required
LOGIN_URL = 'login'
```

## Common Pitfalls

- **Storing passwords in plain text**: Never store raw passwords. Always use `User.objects.create_user()` which hashes the password automatically.
- **Forgetting to include auth middleware**: The `AuthenticationMiddleware` must be in `MIDDLEWARE` for `request.user` to be available.
- **Using GET for logout**: Since Django 5.0, `LogoutView` requires POST requests. Using a simple link (`<a href="/logout/">`) no longer works.

## Best Practices

- **Use Django's built-in auth views** (`LoginView`, `LogoutView`, `PasswordResetView`) instead of writing your own.
- **Set `LOGIN_REDIRECT_URL` and `LOGOUT_REDIRECT_URL`** in settings to control where users go after authentication.
- **Use `include('django.contrib.auth.urls')`** to get all auth URL patterns with a single line.

## Summary

- Django includes a complete **authentication system** with users, groups, and permissions
- The built-in `User` model provides username, email, password hashing, and permission fields
- Use `authenticate()` and `login()` for custom login views, or use built-in `LoginView`
- Include all auth URLs with `path('accounts/', include('django.contrib.auth.urls'))`
- Set `LOGIN_REDIRECT_URL`, `LOGOUT_REDIRECT_URL`, and `LOGIN_URL` in settings

## Code Examples

**Custom login view using Django authenticate() and login() functions**

```python
from django.contrib.auth import authenticate, login
from django.shortcuts import render, redirect

def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            return redirect('home')
    return render(request, 'login.html')
```


## Resources

- [User Authentication in Django](https://docs.djangoproject.com/en/6.0/topics/auth/) — Official guide to Django's authentication system

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*