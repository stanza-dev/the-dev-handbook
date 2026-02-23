---
source_course: "django-foundations"
source_lesson: "django-foundations-cbv-crud"
---

# CRUD with Generic Views

Django's generic editing views make it easy to create, update, and delete objects with minimal code.

## CreateView

Create new objects:

```python
from django.views.generic import CreateView
from django.urls import reverse_lazy
from .models import Question


class QuestionCreateView(CreateView):
    model = Question
    fields = ['question_text', 'pub_date']
    template_name = 'polls/question_form.html'
    success_url = reverse_lazy('polls:index')  # Redirect after success
    
    # Or define success URL dynamically
    def get_success_url(self):
        return reverse('polls:detail', kwargs={'pk': self.object.pk})
```

Why `reverse_lazy`? Because URLs aren't loaded when the class is defined.

```html
<!-- polls/templates/polls/question_form.html -->
<h1>Create Question</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Create</button>
</form>
```

## UpdateView

Edit existing objects:

```python
from django.views.generic import UpdateView


class QuestionUpdateView(UpdateView):
    model = Question
    fields = ['question_text', 'pub_date']
    template_name = 'polls/question_form.html'
    
    def get_success_url(self):
        return reverse('polls:detail', kwargs={'pk': self.object.pk})
```

Same template works for both Create and Update!

```html
<h1>{% if object %}Edit{% else %}Create{% endif %} Question</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Save</button>
</form>
```

## DeleteView

Delete objects with confirmation:

```python
from django.views.generic import DeleteView


class QuestionDeleteView(DeleteView):
    model = Question
    template_name = 'polls/question_confirm_delete.html'
    success_url = reverse_lazy('polls:index')
```

```html
<!-- polls/templates/polls/question_confirm_delete.html -->
<h1>Delete Question</h1>
<p>Are you sure you want to delete "{{ object.question_text }}"?</p>
<form method="post">
    {% csrf_token %}
    <button type="submit">Yes, delete</button>
    <a href="{% url 'polls:detail' object.pk %}">Cancel</a>
</form>
```

## Complete URL Configuration

Here is a full set of URL patterns that wire up all five CRUD views for a single model.

```python
# polls/urls.py
from django.urls import path
from . import views

app_name = 'polls'

urlpatterns = [
    path('', views.QuestionListView.as_view(), name='index'),
    path('<int:pk>/', views.QuestionDetailView.as_view(), name='detail'),
    path('create/', views.QuestionCreateView.as_view(), name='create'),
    path('<int:pk>/edit/', views.QuestionUpdateView.as_view(), name='edit'),
    path('<int:pk>/delete/', views.QuestionDeleteView.as_view(), name='delete'),
]
```

## Using Forms with Generic Views

Use a custom form instead of `fields`:

```python
from .forms import QuestionForm

class QuestionCreateView(CreateView):
    model = Question
    form_class = QuestionForm  # Use custom form
    template_name = 'polls/question_form.html'
    success_url = reverse_lazy('polls:index')
```

## Adding Extra Context

Override `get_context_data()` to pass additional variables to your template beyond the form and object.

```python
class QuestionCreateView(CreateView):
    model = Question
    fields = ['question_text', 'pub_date']
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['page_title'] = 'Create New Question'
        context['categories'] = Category.objects.all()
        return context
```

## Setting Fields on Save

Override `form_valid()` when you need to set fields that are not part of the form, such as the currently logged-in user.

```python
class QuestionCreateView(LoginRequiredMixin, CreateView):
    model = Question
    fields = ['question_text']
    
    def form_valid(self, form):
        form.instance.author = self.request.user  # Set author
        form.instance.pub_date = timezone.now()   # Set date
        return super().form_valid(form)
```

## Common Pitfalls

- **Using `reverse()` instead of `reverse_lazy()` for `success_url`**: At class definition time, URLs are not loaded. Use `reverse_lazy()` for class-level attributes.
- **Not creating the template for DeleteView**: Django expects a `model_confirm_delete.html` template. Without it, you get a `TemplateDoesNotExist` error.
- **Forgetting CSRF token in form templates**: All CBV form templates need `{% csrf_token %}` inside the `<form>` tag, just like function-based views.

## Best Practices

- **Use `form_class` for complex forms** instead of the `fields` attribute when you need custom validation or widgets.
- **Override `form_valid()`** to add extra data (like `request.user`) before saving.
- **Combine with `LoginRequiredMixin`**: Place it before the generic view in the class definition (e.g., `class MyView(LoginRequiredMixin, CreateView)`).

## Summary

- `CreateView` renders a form and saves new objects on valid POST submission
- `UpdateView` pre-fills a form with existing object data for editing
- `DeleteView` shows a confirmation page and deletes the object on POST
- Use `reverse_lazy()` (not `reverse()`) for `success_url` at class level
- Override `form_valid()` to add custom logic before saving

## Code Examples

**Complete CRUD with CreateView, UpdateView, and DeleteView generic editing views**

```python
from django.views.generic import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Question

class QuestionCreateView(CreateView):
    model = Question
    fields = ['question_text', 'pub_date']
    success_url = reverse_lazy('polls:index')

class QuestionUpdateView(UpdateView):
    model = Question
    fields = ['question_text', 'pub_date']

class QuestionDeleteView(DeleteView):
    model = Question
    success_url = reverse_lazy('polls:index')
```


## Resources

- [Generic Editing Views](https://docs.djangoproject.com/en/6.0/ref/class-based-views/generic-editing/) — Reference for CreateView, UpdateView, and DeleteView

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*