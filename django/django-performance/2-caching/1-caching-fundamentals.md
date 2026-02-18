---
source_course: "django-performance"
source_lesson: "django-performance-caching-fundamentals"
---

# Django Caching Fundamentals

Caching stores computed results to avoid redundant processing.

## Cache Backends

```python
# settings.py

# Memory cache (development)
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}

# Redis cache (production)
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        },
        'KEY_PREFIX': 'mysite',
    }
}

# Database cache
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'cache_table',
    }
}
# Run: python manage.py createcachetable
```

## Low-Level Cache API

```python
from django.core.cache import cache

# Set cache
cache.set('my_key', 'my_value', timeout=300)  # 5 minutes

# Get cache
value = cache.get('my_key')  # Returns None if not found
value = cache.get('my_key', 'default_value')

# Get or set
def get_expensive_data():
    # Expensive computation
    return compute_something()

value = cache.get_or_set('expensive_key', get_expensive_data, timeout=3600)

# Delete
cache.delete('my_key')

# Clear all
cache.clear()

# Multiple keys
cache.set_many({'key1': 'value1', 'key2': 'value2'}, timeout=300)
values = cache.get_many(['key1', 'key2'])
cache.delete_many(['key1', 'key2'])

# Increment/Decrement
cache.set('counter', 1)
cache.incr('counter')  # 2
cache.decr('counter')  # 1
```

## Caching in Views

```python
from django.views.decorators.cache import cache_page
from django.core.cache import cache


@cache_page(60 * 15)  # Cache for 15 minutes
def article_list(request):
    articles = Article.objects.published()
    return render(request, 'articles/list.html', {'articles': articles})


def article_detail(request, slug):
    cache_key = f'article_{slug}'
    article = cache.get(cache_key)
    
    if article is None:
        article = get_object_or_404(Article, slug=slug)
        cache.set(cache_key, article, timeout=3600)
    
    return render(request, 'articles/detail.html', {'article': article})
```

## Template Fragment Caching

```html
{% load cache %}

{# Cache for 500 seconds #}
{% cache 500 sidebar %}
    <div class="sidebar">
        {% for item in sidebar_items %}
            <p>{{ item.title }}</p>
        {% endfor %}
    </div>
{% endcache %}

{# Cache per user #}
{% cache 500 sidebar request.user.id %}
    {# User-specific sidebar #}
{% endcache %}

{# Multiple vary keys #}
{% cache 500 article article.pk article.updated_at %}
    {# Invalidates when article is updated #}
{% endcache %}
```

## Per-Site Cache

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.cache.UpdateCacheMiddleware',    # First
    'django.middleware.common.CommonMiddleware',
    # ... other middleware ...
    'django.middleware.cache.FetchFromCacheMiddleware',  # Last
]

CACHE_MIDDLEWARE_ALIAS = 'default'
CACHE_MIDDLEWARE_SECONDS = 600
CACHE_MIDDLEWARE_KEY_PREFIX = 'mysite'
```

## Cache Invalidation

```python
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from django.core.cache import cache


@receiver([post_save, post_delete], sender=Article)
def invalidate_article_cache(sender, instance, **kwargs):
    """Clear article cache when article changes."""
    cache.delete(f'article_{instance.slug}')
    cache.delete('article_list')
    cache.delete('homepage_articles')


class Article(models.Model):
    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        # Invalidate related caches
        self.invalidate_cache()
    
    def invalidate_cache(self):
        cache.delete_many([
            f'article_{self.slug}',
            f'author_{self.author_id}_articles',
            f'category_{self.category_id}_articles',
        ])
```

## Resources

- [Django Cache Framework](https://docs.djangoproject.com/en/6.0/topics/cache/) — Complete caching documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*