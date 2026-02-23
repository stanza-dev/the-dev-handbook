---
source_course: "django-foundations"
source_lesson: "django-foundations-cbv-mixins"
---

# View Mixins and Composition

## Introduction

Mixins are small, reusable classes that add specific functionality to your views through multiple inheritance. Django's class-based views are designed around mixins, letting you compose complex views from simple building blocks.

## Key Concepts

- **Mixin**: A class that provides specific functionality but is not meant to be used on its own.
- **Multiple Inheritance**: Python's ability to inherit from multiple classes, which Django uses to compose views.
- **Method Resolution Order (MRO)**: The order in which Python resolves method calls across multiple parent classes.

## Real World Context

As your application grows, you will find yourself repeating the same patterns across views: checking permissions, adding common context data, handling AJAX requests, or logging actions. Mixins let you extract these patterns into reusable components that can be added to any view.

## Deep Dive

### Django's Built-in Mixins

Django provides several authentication mixins:

```python
from django.contrib.auth.mixins import (
    LoginRequiredMixin,
    PermissionRequiredMixin,
    UserPassesTestMixin,
)
from django.views.generic import ListView, CreateView


class ProtectedListView(LoginRequiredMixin, ListView):
    """Only logged-in users can see this."""
    model = Question
    login_url = '/accounts/login/'
    redirect_field_name = 'next'


class StaffCreateView(PermissionRequiredMixin, CreateView):
    """Only users with the permission can create."""
    model = Question
    fields = ['question_text', 'pub_date']
    permission_required = 'polls.add_question'


class OwnerOnlyView(UserPassesTestMixin, DetailView):
    """Only the object owner can view."""
    model = Question

    def test_func(self):
        obj = self.get_object()
        return obj.author == self.request.user
```

### Creating Custom Mixins

Build your own mixins for reusable behavior:

```python
class TitleMixin:
    """Adds a title to the template context."""
    title = ''

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['page_title'] = self.title
        return context


class QuestionListView(TitleMixin, ListView):
    model = Question
    title = 'All Questions'
```

```python
class AjaxResponseMixin:
    """Return JSON for AJAX requests, HTML otherwise."""

    def dispatch(self, request, *args, **kwargs):
        self.is_ajax = request.headers.get(
            'X-Requested-With'
        ) == 'XMLHttpRequest'
        return super().dispatch(request, *args, **kwargs)

    def render_to_response(self, context, **kwargs):
        if self.is_ajax:
            return JsonResponse(self.get_ajax_data(context))
        return super().render_to_response(context, **kwargs)

    def get_ajax_data(self, context):
        return {'status': 'ok'}
```

### Mixin Ordering Matters

Always place mixins **before** the base view class:

```python
# CORRECT: Mixin before base view
class MyView(LoginRequiredMixin, TitleMixin, ListView):
    pass

# WRONG: Base view before mixin
# class MyView(ListView, LoginRequiredMixin):
#     pass  # LoginRequiredMixin may not work properly!
```

Python's MRO processes classes left to right. Mixins should intercept the method call before it reaches the base view.

### Composing Multiple Mixins

You can combine multiple mixins into a single base class, then reuse that base class across several views.

```python
class StaffRequiredMixin(LoginRequiredMixin, UserPassesTestMixin):
    """Combined mixin: must be logged in AND staff."""
    def test_func(self):
        return self.request.user.is_staff


class AdminDashboardView(StaffRequiredMixin, TitleMixin, TemplateView):
    template_name = 'admin/dashboard.html'
    title = 'Admin Dashboard'
```

### FormValidMessageMixin Example

This custom mixin displays a success message after a form is saved, using Django's messages framework.

```python
from django.contrib import messages

class FormValidMessageMixin:
    """Display a success message after valid form submission."""
    success_message = ''

    def form_valid(self, form):
        response = super().form_valid(form)
        if self.success_message:
            messages.success(self.request, self.success_message)
        return response


class QuestionCreateView(FormValidMessageMixin, CreateView):
    model = Question
    fields = ['question_text', 'pub_date']
    success_message = 'Question created successfully!'
    success_url = reverse_lazy('polls:index')
```

## Common Pitfalls

- **Wrong mixin order**: Mixins must come before the base view class (e.g., `LoginRequiredMixin, ListView`, not `ListView, LoginRequiredMixin`). Wrong order breaks the method resolution.
- **Forgetting `super()` calls**: Every mixin method that overrides a parent method must call `super()` to ensure the chain of mixins all execute properly.
- **Making mixins too complex**: Mixins should do one thing well. If a mixin grows beyond a single responsibility, split it into multiple smaller mixins.

## Best Practices

- **Keep mixins small and focused**: Each mixin should add exactly one piece of functionality.
- **Always call `super()`** in overridden methods to preserve the mixin chain.
- **Document mixin requirements**: If a mixin expects certain attributes (like `model` or `title`), document them clearly.

## Summary

- Mixins add **reusable functionality** to class-based views through multiple inheritance
- Django provides `LoginRequiredMixin`, `PermissionRequiredMixin`, and `UserPassesTestMixin` for access control
- Custom mixins can add context data, handle AJAX, show messages, or enforce business rules
- Always place mixins **before** the base view class in the inheritance list
- Call `super()` in every overridden method to maintain the full mixin chain

## Code Examples

**Composing a view with LoginRequiredMixin, a custom TitleMixin, and ListView**

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView

class TitleMixin:
    title = ''
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['page_title'] = self.title
        return context

class QuestionListView(LoginRequiredMixin, TitleMixin, ListView):
    model = Question
    title = 'All Questions'
    paginate_by = 10
```


## Resources

- [Using Mixins with Class-Based Views](https://docs.djangoproject.com/en/6.0/topics/class-based-views/mixins/) — Official guide to using mixins with Django class-based views

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*