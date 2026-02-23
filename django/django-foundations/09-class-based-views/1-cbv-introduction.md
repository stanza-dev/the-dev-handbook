---
source_course: "django-foundations"
source_lesson: "django-foundations-cbv-introduction"
---

# Introduction to Class-Based Views

Class-Based Views (CBVs) are an alternative to function-based views that use Python classes. They provide inheritance, mixins, and built-in patterns for common tasks.

## Why Class-Based Views?

| Function-Based | Class-Based |
|----------------|-------------|
| Simple, explicit | Reusable, extendable |
| Good for unique logic | Good for common patterns |
| All code in one place | Logic split into methods |
| Direct flow | Method dispatch |

## Basic Class-Based View

A class-based view defines separate methods for each HTTP method. Django's `View` base class routes requests to the appropriate method automatically.

```python
# polls/views.py
from django.http import HttpResponse
from django.views import View


class HomeView(View):
    def get(self, request):
        return HttpResponse("Hello from GET!")
    
    def post(self, request):
        return HttpResponse("Hello from POST!")
```

## URL Configuration

Class-based views must be converted to callable functions using `.as_view()` before they can be used in URL patterns.

```python
# polls/urls.py
from django.urls import path
from .views import HomeView

urlpatterns = [
    path('', HomeView.as_view(), name='home'),
]
```

Use `.as_view()` to convert the class to a view function.

## TemplateView

Render a template without custom logic:

```python
from django.views.generic import TemplateView


class AboutView(TemplateView):
    template_name = 'about.html'
    
    # Optional: add context
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['page_title'] = 'About Us'
        return context
```

Or directly in URLs:

```python
urlpatterns = [
    path('about/', TemplateView.as_view(template_name='about.html'), name='about'),
]
```

## ListView

Display a list of objects:

```python
from django.views.generic import ListView
from .models import Question


class QuestionListView(ListView):
    model = Question
    template_name = 'polls/question_list.html'  # Default: polls/question_list.html
    context_object_name = 'questions'            # Default: object_list
    ordering = ['-pub_date']
    paginate_by = 10  # Enable pagination
    
    def get_queryset(self):
        """Custom filtering."""
        return Question.objects.filter(is_active=True)
```

```html
<!-- polls/templates/polls/question_list.html -->
<h1>Questions</h1>
<ul>
{% for question in questions %}
    <li>{{ question.question_text }}</li>
{% empty %}
    <li>No questions available.</li>
{% endfor %}
</ul>

<!-- Pagination -->
{% if is_paginated %}
<nav>
    {% if page_obj.has_previous %}
        <a href="?page={{ page_obj.previous_page_number }}">Previous</a>
    {% endif %}
    
    Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}
    
    {% if page_obj.has_next %}
        <a href="?page={{ page_obj.next_page_number }}">Next</a>
    {% endif %}
</nav>
{% endif %}
```

## DetailView

Display a single object:

```python
from django.views.generic import DetailView


class QuestionDetailView(DetailView):
    model = Question
    template_name = 'polls/question_detail.html'
    context_object_name = 'question'
```

```python
# urls.py
urlpatterns = [
    path('<int:pk>/', QuestionDetailView.as_view(), name='detail'),
    # Or use slug
    path('<slug:slug>/', QuestionDetailView.as_view(), name='detail'),
]
```

```html
<!-- polls/templates/polls/question_detail.html -->
<h1>{{ question.question_text }}</h1>
<p>Published: {{ question.pub_date }}</p>

<h2>Choices:</h2>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} - {{ choice.votes }} votes</li>
{% endfor %}
</ul>
```

## Common Pitfalls

- **Forgetting `.as_view()` in URL patterns**: Class-based views must be called with `.as_view()` in `urlpatterns`. Without it, Django raises a `TypeError`.
- **Confusing `template_name` defaults**: If you don't set `template_name`, Django expects `app/model_list.html` for ListView and `app/model_detail.html` for DetailView.
- **Not calling `super()` in overridden methods**: When overriding `get_context_data()` or `get_queryset()`, always call `super()` first to preserve the default behavior.

## Best Practices

- **Set `context_object_name`** for readable template variable names instead of the default `object_list`.
- **Override `get_queryset()`** for custom filtering instead of hardcoding queries.
- **Use `paginate_by`** on ListViews to prevent loading too many objects at once.

## Summary

- Class-based views use Python classes with methods for each HTTP method (`get()`, `post()`)
- `TemplateView`, `ListView`, and `DetailView` are the most common generic display views
- Use `.as_view()` in URL patterns to convert a class into a callable view function
- Override `get_queryset()` for custom filtering and `get_context_data()` for extra context
- CBVs support inheritance and mixins for code reuse across views

## Code Examples

**A ListView class-based view with pagination and custom ordering**

```python
from django.views.generic import ListView
from .models import Question

class QuestionListView(ListView):
    model = Question
    template_name = 'polls/question_list.html'
    context_object_name = 'questions'
    ordering = ['-pub_date']
    paginate_by = 10

# urls.py
path('', QuestionListView.as_view(), name='index')
```


## Resources

- [Class-based Views](https://docs.djangoproject.com/en/6.0/topics/class-based-views/) — Official introduction to class-based views

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*