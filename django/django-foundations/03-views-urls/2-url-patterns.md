---
source_course: "django-foundations"
source_lesson: "django-foundations-url-patterns"
---

# Advanced URL Patterns

Django's URL dispatcher is powerful and flexible. Let's explore advanced patterns and best practices.

## Naming URLs

Always name your URLs for easy referencing:

```python
# polls/urls.py
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:pk>/', views.detail, name='detail'),
]
```

Use names in templates and code:

```python
# In views
from django.urls import reverse

url = reverse('detail', args=[5])  # '/polls/5/'
url = reverse('detail', kwargs={'pk': 5})  # '/polls/5/'
```

```html
<!-- In templates -->
<a href="{% url 'detail' question.id %}">View Details</a>
```

## App Namespacing

When you have multiple apps, use namespaces:

```python
# polls/urls.py
from django.urls import path
from . import views

app_name = 'polls'  # Add namespace

urlpatterns = [
    path('', views.index, name='index'),
    path('<int:pk>/', views.detail, name='detail'),
]
```

Now reference with namespace:

```python
reverse('polls:detail', args=[5])
```

```html
<a href="{% url 'polls:detail' question.id %}">View</a>
```

## Including Other URLconfs

Organize URLs across apps:

```python
# mysite/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
    path('blog/', include('blog.urls')),
    path('api/', include('api.urls')),
]
```

## Regular Expression Patterns

For complex patterns, use `re_path`:

```python
from django.urls import path, re_path
from . import views

urlpatterns = [
    # Standard path
    path('articles/<int:year>/', views.year_archive),
    
    # Regex pattern for 4-digit year
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    
    # Regex for valid usernames
    re_path(r'^user/(?P<username>[\w.@+-]+)/$', views.user_profile),
]
```

## Passing Extra Options

Pass extra context to views:

```python
urlpatterns = [
    path(
        'blog/<int:year>/',
        views.year_archive,
        {'foo': 'bar'},  # Extra context
        name='year_archive'
    ),
]

# View receives: request, year, foo='bar'
def year_archive(request, year, foo):
    pass
```

## Custom Path Converters

Create your own converters:

```python
# converters.py
class FourDigitYearConverter:
    regex = '[0-9]{4}'

    def to_python(self, value):
        return int(value)

    def to_url(self, value):
        return '%04d' % value

# urls.py
from django.urls import path, register_converter
from . import converters, views

register_converter(converters.FourDigitYearConverter, 'yyyy')

urlpatterns = [
    path('articles/<yyyy:year>/', views.year_archive),
]
```

## URL Best Practices

1. **Use meaningful names**: `article-detail` not `view1`
2. **Be consistent**: Pick a pattern and stick with it
3. **Use namespaces**: Prevent name collisions between apps
4. **Keep URLs clean**: No file extensions, no query params for navigation
5. **Use hyphens**: `my-article` not `my_article` or `myArticle`

```python
# Good URL design
/articles/
/articles/2024/
/articles/2024/my-first-post/
/users/johndoe/

# Bad URL design
/showArticle.php?id=123
/articles_list/
/get_user?username=johndoe
```

## Resources

- [URL Dispatcher](https://docs.djangoproject.com/en/6.0/topics/http/urls/) — Official guide to Django's URL dispatcher

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*