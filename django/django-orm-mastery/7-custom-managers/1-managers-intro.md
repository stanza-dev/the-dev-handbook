---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-custom-managers-intro"
---

# Custom Model Managers

Managers are the interface through which database query operations are provided to Django models. You can customize them to add reusable query methods and change default behavior.

## Default Manager

Every model has at least one manager called `objects`:

```python
# Default manager
Article.objects.all()
Article.objects.filter(status='published')
```

## Creating a Custom Manager

```python
from django.db import models


class ArticleManager(models.Manager):
    """Custom manager with additional methods."""
    
    def published(self):
        """Return only published articles."""
        return self.filter(status='published', pub_date__lte=timezone.now())
    
    def drafts(self):
        """Return draft articles."""
        return self.filter(status='draft')
    
    def by_author(self, author):
        """Return articles by a specific author."""
        return self.filter(author=author)
    
    def popular(self, min_views=1000):
        """Return articles with many views."""
        return self.filter(views__gte=min_views)
    
    def recent(self, days=7):
        """Return articles from the last N days."""
        cutoff = timezone.now() - timedelta(days=days)
        return self.filter(pub_date__gte=cutoff)


class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20, default='draft')
    pub_date = models.DateTimeField(null=True, blank=True)
    views = models.IntegerField(default=0)
    author = models.ForeignKey('Author', on_delete=models.CASCADE)
    
    # Use custom manager
    objects = ArticleManager()
```

## Using Custom Manager Methods

```python
# Chainable methods
Article.objects.published()  # QuerySet of published articles
Article.objects.published().filter(category='tech')  # Chain with filter
Article.objects.by_author(user).published()  # Chain custom methods

# Combine with other QuerySet methods
Article.objects.popular().order_by('-pub_date')[:10]
```

## Modifying Default QuerySet

Change what `all()` returns:

```python
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status='published')


class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    
    # Default manager (includes all)
    objects = models.Manager()
    
    # Alternative manager (only published)
    published = PublishedManager()
```

```python
# Uses different managers
Article.objects.all()     # All articles
Article.published.all()   # Only published
```

## Multiple Managers

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    
    # First manager is the default
    objects = ArticleManager()  # Default
    published = PublishedManager()
    drafts = DraftManager()
    
    class Meta:
        # Explicitly set default manager
        default_manager_name = 'objects'
```

## Manager with Initial QuerySet Filtering

```python
class SoftDeleteManager(models.Manager):
    """Manager that excludes soft-deleted objects."""
    
    def get_queryset(self):
        return super().get_queryset().filter(deleted_at__isnull=True)
    
    def with_deleted(self):
        """Include soft-deleted objects."""
        return super().get_queryset()
    
    def deleted_only(self):
        """Only soft-deleted objects."""
        return super().get_queryset().filter(deleted_at__isnull=False)


class Document(models.Model):
    title = models.CharField(max_length=200)
    deleted_at = models.DateTimeField(null=True, blank=True)
    
    objects = SoftDeleteManager()
    all_objects = models.Manager()  # Bypass soft delete
```

## Manager Methods with Complex Logic

```python
class OrderManager(models.Manager):
    def pending_review(self):
        """Orders that need review."""
        return self.filter(
            status='submitted',
            total__gte=1000
        ).select_related('customer')
    
    def for_shipping(self):
        """Orders ready to ship."""
        return self.filter(
            status='paid',
            items__in_stock=True
        ).distinct().prefetch_related('items')
    
    def revenue_by_month(self):
        """Aggregate revenue by month."""
        from django.db.models.functions import TruncMonth
        return self.filter(
            status='completed'
        ).annotate(
            month=TruncMonth('completed_at')
        ).values('month').annotate(
            total=Sum('total'),
            count=Count('id')
        ).order_by('month')
```

## Resources

- [Managers](https://docs.djangoproject.com/en/6.0/topics/db/managers/) — Official guide to model managers

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*