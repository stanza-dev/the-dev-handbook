---
source_course: "django-foundations"
source_lesson: "django-foundations-function-based-views"
---

# Function-Based Views

Views are Python functions (or classes) that receive HTTP requests and return HTTP responses. They're the heart of your application's logic.

## Basic View Structure

A view function:
1. Receives an `HttpRequest` object
2. Does some work (query database, process data, etc.)
3. Returns an `HttpResponse` object

```python
# polls/views.py
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world!")
```

## Views with Database Queries

Let's create more useful views:

```python
# polls/views.py
from django.http import HttpResponse
from .models import Question


def index(request):
    latest_questions = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_questions])
    return HttpResponse(output)


def detail(request, question_id):
    question = Question.objects.get(pk=question_id)
    return HttpResponse(f"Question: {question.question_text}")


def results(request, question_id):
    return HttpResponse(f"Results for question {question_id}")


def vote(request, question_id):
    return HttpResponse(f"Voting on question {question_id}")
```

## URL Configuration

Map URLs to views in `urls.py`:

```python
# polls/urls.py
from django.urls import path
from . import views

urlpatterns = [
    # /polls/
    path('', views.index, name='index'),
    # /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

## URL Path Converters

Django provides path converters to capture URL parts:

| Converter | Description | Example |
|-----------|-------------|----------|
| `str` | Any non-empty string (default) | `<str:slug>` → `"hello-world"` |
| `int` | Zero or positive integer | `<int:id>` → `42` |
| `slug` | ASCII letters, numbers, hyphens, underscores | `<slug:title>` → `"my-post"` |
| `uuid` | UUID format | `<uuid:id>` → `"075194d3-..."` |
| `path` | Any string including `/` | `<path:file>` → `"dir/file.txt"` |

```python
# Examples
path('articles/<int:year>/', views.year_archive),
path('articles/<int:year>/<int:month>/', views.month_archive),
path('articles/<int:year>/<int:month>/<slug:slug>/', views.article_detail),
```

## Handling Errors

What if an object doesn't exist?

```python
from django.http import Http404

def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return HttpResponse(f"Question: {question.question_text}")
```

## Shortcut: get_object_or_404

Django provides a helpful shortcut:

```python
from django.shortcuts import get_object_or_404

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return HttpResponse(f"Question: {question.question_text}")
```

Also available: `get_list_or_404()` for QuerySets.

## Request Object

The `request` object contains information about the HTTP request:

```python
def my_view(request):
    # HTTP method
    method = request.method  # 'GET', 'POST', etc.
    
    # GET parameters (?name=value)
    name = request.GET.get('name', 'default')
    
    # POST data
    if request.method == 'POST':
        username = request.POST.get('username')
    
    # Request path
    path = request.path  # '/polls/5/'
    
    # User (if authenticated)
    user = request.user
    
    # Is AJAX request?
    is_ajax = request.headers.get('X-Requested-With') == 'XMLHttpRequest'
```

## Resources

- [Writing Views](https://docs.djangoproject.com/en/6.0/topics/http/views/) — Official guide to writing Django views

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*