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

```python
class QuestionCreateView(LoginRequiredMixin, CreateView):
    model = Question
    fields = ['question_text']
    
    def form_valid(self, form):
        form.instance.author = self.request.user  # Set author
        form.instance.pub_date = timezone.now()   # Set date
        return super().form_valid(form)
```

## Resources

- [Generic Editing Views](https://docs.djangoproject.com/en/6.0/ref/class-based-views/generic-editing/) — Reference for CreateView, UpdateView, and DeleteView

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*