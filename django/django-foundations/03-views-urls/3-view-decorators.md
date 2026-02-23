---
source_course: "django-foundations"
source_lesson: "django-foundations-view-decorators"
---

# View Decorators

## Introduction

Django provides a powerful set of decorators that let you modify view behavior without changing the view function itself. These decorators handle common patterns like restricting HTTP methods, requiring authentication, and controlling caching.

## Key Concepts

- **Decorator**: A function that wraps another function to add behavior before or after execution.
- **HTTP Method Restriction**: Limiting which HTTP methods (GET, POST, etc.) a view accepts.
- **View Decoration**: Applying decorators to views in `views.py` or directly in `urls.py`.

## Real World Context

In production applications, you almost never want a view to accept all HTTP methods. A page that displays data should only respond to GET, while a form submission handler should only respond to POST. Decorators enforce these constraints cleanly.

## Deep Dive

Django offers several built-in view decorators in `django.views.decorators`.

### HTTP Method Decorators

Restrict which HTTP methods a view accepts:

```python
from django.views.decorators.http import require_http_methods, require_GET, require_POST, require_safe

@require_GET
def article_list(request):
    """Only accepts GET requests."""
    articles = Article.objects.all()
    return render(request, 'articles/list.html', {'articles': articles})

@require_POST
def delete_article(request, pk):
    """Only accepts POST requests."""
    article = get_object_or_404(Article, pk=pk)
    article.delete()
    return redirect('article_list')

@require_http_methods(["GET", "POST"])
def edit_article(request, pk):
    """Accepts GET and POST only."""
    article = get_object_or_404(Article, pk=pk)
    if request.method == 'POST':
        form = ArticleForm(request.POST, instance=article)
        if form.is_valid():
            form.save()
            return redirect('article_detail', pk=pk)
    else:
        form = ArticleForm(instance=article)
    return render(request, 'articles/edit.html', {'form': form})

@require_safe
def public_data(request):
    """Only accepts GET and HEAD (safe methods)."""
    return JsonResponse({'status': 'ok'})
```

If a disallowed method is used, Django returns `405 Method Not Allowed`.

### CSRF Decorators

Control CSRF protection on specific views:

```python
from django.views.decorators.csrf import csrf_exempt, csrf_protect

@csrf_exempt
def webhook_handler(request):
    """External webhooks can't provide CSRF tokens."""
    payload = json.loads(request.body)
    process_webhook(payload)
    return JsonResponse({'received': True})

@csrf_protect
def special_form(request):
    """Ensure CSRF protection even if middleware is disabled."""
    pass
```

### Caching Decorators

Control response caching:

```python
from django.views.decorators.cache import cache_page, never_cache

@cache_page(60 * 15)  # Cache for 15 minutes
def article_list(request):
    articles = Article.objects.all()
    return render(request, 'articles/list.html', {'articles': articles})

@never_cache
def dashboard(request):
    """Always fetch fresh data."""
    return render(request, 'dashboard.html')
```

### Stacking Multiple Decorators

Decorators are applied bottom-up (closest to the function runs first):

```python
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_POST

@login_required
@require_POST
def vote(request, question_id):
    """Must be logged in AND use POST."""
    question = get_object_or_404(Question, pk=question_id)
    # Process vote...
    return redirect('polls:results', pk=question_id)
```

### Decorating Class-Based Views

Use `method_decorator` for CBVs:

```python
from django.utils.decorators import method_decorator
from django.views.decorators.cache import cache_page
from django.views.generic import ListView

@method_decorator(cache_page(60 * 15), name='dispatch')
class ArticleListView(ListView):
    model = Article
```

Or apply directly in URLs:

```python
from django.contrib.auth.decorators import login_required

urlpatterns = [
    path('vote/', login_required(VoteView.as_view()), name='vote'),
]
```

## Common Pitfalls

- **Using `@csrf_exempt` carelessly**: Only use it for views that genuinely cannot provide CSRF tokens, like external API webhooks. Never apply it to forms used by your own frontend.
- **Stacking decorators in wrong order**: Decorators apply bottom-up. `@login_required` should be the outermost (top) decorator so authentication is checked first.
- **Forgetting `method_decorator` for CBVs**: Regular decorators do not work directly on class methods. Use `method_decorator` or apply decorators in URLconf.

## Best Practices

- **Always restrict HTTP methods**: Use `@require_GET`, `@require_POST`, or `@require_http_methods` on every view to enforce the expected method.
- **Prefer `@require_safe` over `@require_GET`**: `require_safe` allows both GET and HEAD, which is more correct for read-only views.
- **Apply `@login_required` at the outermost level**: Check authentication before checking methods or other conditions.

## Summary

- `@require_GET`, `@require_POST`, and `@require_http_methods` restrict allowed HTTP methods, returning 405 for others
- `@csrf_exempt` disables CSRF protection for specific views like external webhooks
- `@cache_page(seconds)` caches view responses for improved performance
- Decorators are stacked bottom-up, with the outermost running first
- Use `method_decorator` to apply function decorators to class-based views

## Code Examples

**Stacking login_required and require_http_methods decorators to protect a view**

```python
from django.views.decorators.http import require_http_methods
from django.contrib.auth.decorators import login_required

@login_required
@require_http_methods(["GET", "POST"])
def edit_article(request, pk):
    article = get_object_or_404(Article, pk=pk)
    if request.method == 'POST':
        form = ArticleForm(request.POST, instance=article)
        if form.is_valid():
            form.save()
            return redirect('article_detail', pk=pk)
    else:
        form = ArticleForm(instance=article)
    return render(request, 'articles/edit.html', {'form': form})
```


## Resources

- [View Decorators](https://docs.djangoproject.com/en/6.0/topics/http/decorators/) — Official reference for Django view decorators

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*