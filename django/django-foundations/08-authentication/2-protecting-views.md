---
source_course: "django-foundations"
source_lesson: "django-foundations-protecting-views"
---

# Protecting Views

Django provides several ways to restrict access to views based on authentication status or permissions.

## The login_required Decorator

Require users to be logged in:

```python
from django.contrib.auth.decorators import login_required

@login_required
def profile(request):
    return render(request, 'profile.html')
```

If not logged in, users are redirected to `LOGIN_URL` (default: `/accounts/login/`).

### Custom Redirect

```python
@login_required(login_url='/custom/login/')
def profile(request):
    pass

# Or set redirect field name
@login_required(redirect_field_name='return_to')
def profile(request):
    pass
```

## Permission Required

Check for specific permissions:

```python
from django.contrib.auth.decorators import permission_required

@permission_required('polls.add_question')
def create_question(request):
    pass

# Multiple permissions
@permission_required(['polls.add_question', 'polls.change_question'])
def manage_questions(request):
    pass

# Raise 403 instead of redirecting
@permission_required('polls.delete_question', raise_exception=True)
def delete_question(request, pk):
    pass
```

## User Passes Test

Custom logic for access control:

```python
from django.contrib.auth.decorators import user_passes_test

def is_staff(user):
    return user.is_staff

@user_passes_test(is_staff)
def staff_dashboard(request):
    pass

# Using lambda
@user_passes_test(lambda u: u.is_superuser)
def admin_only(request):
    pass
```

## Checking in Templates

```html
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}!</p>
    <a href="{% url 'logout' %}">Logout</a>
{% else %}
    <a href="{% url 'login' %}">Login</a>
{% endif %}

{% if user.is_staff %}
    <a href="{% url 'admin:index' %}">Admin</a>
{% endif %}

{% if perms.polls.add_question %}
    <a href="{% url 'create_question' %}">Create Question</a>
{% endif %}
```

## Checking in Views

```python
def my_view(request):
    # Check authentication
    if not request.user.is_authenticated:
        return redirect('login')
    
    # Check permissions
    if not request.user.has_perm('polls.change_question'):
        return HttpResponseForbidden("Permission denied")
    
    # Check multiple permissions
    if request.user.has_perms(['polls.add_question', 'polls.delete_question']):
        # User has both permissions
        pass
    
    return render(request, 'page.html')
```

## LoginRequiredMixin for Class-Based Views

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin
from django.views.generic import ListView

class QuestionListView(LoginRequiredMixin, ListView):
    model = Question
    login_url = '/login/'  # Optional: override LOGIN_URL
    redirect_field_name = 'next'


class QuestionCreateView(PermissionRequiredMixin, CreateView):
    model = Question
    permission_required = 'polls.add_question'
    # Or multiple: permission_required = ['polls.add_question', 'polls.view_question']
```

## Creating a User Registration View

```python
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth import login

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)  # Log in the new user
            return redirect('home')
    else:
        form = UserCreationForm()
    
    return render(request, 'registration/register.html', {'form': form})
```

```html
<!-- templates/registration/register.html -->
<h1>Register</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Create Account</button>
</form>
<p>Already have an account? <a href="{% url 'login' %}">Login</a></p>
```

## Resources

- [Using the Django Authentication System](https://docs.djangoproject.com/en/6.0/topics/auth/default/) — Complete guide to using Django's authentication features

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*