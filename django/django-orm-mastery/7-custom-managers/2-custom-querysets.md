---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-custom-querysets"
---

# Custom QuerySets

Custom QuerySets allow you to define chainable methods that can be used both from the manager AND from other queryset methods.

## The Problem with Manager Methods

```python
# Manager method
class ArticleManager(models.Manager):
    def published(self):
        return self.filter(status='published')

# This works:
Article.objects.published()

# But this doesn't work:
Article.objects.filter(category='tech').published()  # AttributeError!
```

## Solution: Custom QuerySet

```python
from django.db import models


class ArticleQuerySet(models.QuerySet):
    """Custom QuerySet with chainable methods."""
    
    def published(self):
        return self.filter(status='published', pub_date__lte=timezone.now())
    
    def drafts(self):
        return self.filter(status='draft')
    
    def featured(self):
        return self.filter(is_featured=True)
    
    def by_author(self, author):
        return self.filter(author=author)
    
    def popular(self, min_views=1000):
        return self.filter(views__gte=min_views)


class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    is_featured = models.BooleanField(default=False)
    views = models.IntegerField(default=0)
    pub_date = models.DateTimeField(null=True)
    author = models.ForeignKey('Author', on_delete=models.CASCADE)
    
    # Use QuerySet as manager
    objects = ArticleQuerySet.as_manager()
```

## Chainable Methods

Now all methods are fully chainable:

```python
# All of these work!
Article.objects.published()
Article.objects.published().featured()
Article.objects.filter(category='tech').published()
Article.objects.by_author(user).published().popular()
Article.objects.all().published().order_by('-pub_date')[:10]
```

## QuerySet + Manager Together

For both manager and queryset methods:

```python
class ArticleQuerySet(models.QuerySet):
    """Chainable filter methods."""
    
    def published(self):
        return self.filter(status='published')
    
    def featured(self):
        return self.filter(is_featured=True)


class ArticleManager(models.Manager):
    """Manager with aggregate methods."""
    
    def get_queryset(self):
        return ArticleQuerySet(self.model, using=self._db)
    
    # Expose QuerySet methods
    def published(self):
        return self.get_queryset().published()
    
    def featured(self):
        return self.get_queryset().featured()
    
    # Manager-only methods (non-chainable)
    def statistics(self):
        return self.aggregate(
            total=Count('id'),
            total_views=Sum('views'),
            avg_views=Avg('views')
        )
    
    def create_draft(self, **kwargs):
        return self.create(status='draft', **kwargs)


class Article(models.Model):
    # ... fields ...
    objects = ArticleManager()
```

## from_queryset() Shortcut

```python
class ArticleQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status='published')
    
    def featured(self):
        return self.filter(is_featured=True)


class ArticleManager(models.Manager):
    def statistics(self):
        return self.aggregate(total=Count('id'))


# Combine Manager and QuerySet
ArticleManager = ArticleManager.from_queryset(ArticleQuerySet)

class Article(models.Model):
    objects = ArticleManager()
```

## Practical Example: Blog Application

```python
class PostQuerySet(models.QuerySet):
    def published(self):
        return self.filter(
            status='published',
            pub_date__lte=timezone.now()
        )
    
    def by_tag(self, tag_slug):
        return self.filter(tags__slug=tag_slug)
    
    def by_category(self, category_slug):
        return self.filter(category__slug=category_slug)
    
    def search(self, query):
        return self.filter(
            Q(title__icontains=query) |
            Q(content__icontains=query) |
            Q(excerpt__icontains=query)
        )
    
    def with_comment_count(self):
        return self.annotate(comment_count=Count('comments'))
    
    def optimized(self):
        """Common optimizations for list views."""
        return self.select_related(
            'author', 'category'
        ).prefetch_related(
            'tags'
        ).with_comment_count()


class Post(models.Model):
    # ... fields ...
    objects = PostQuerySet.as_manager()
```

Usage:
```python
# Highly readable, reusable queries
posts = Post.objects.published().by_category('python').optimized()[:10]

results = Post.objects.published().search('django').with_comment_count()
```\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test custom querysets with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some custom querysets features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- Custom QuerySets is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Custom QuerySet with chainable methods exposed via as_manager()**

```python
from django.db import models

class ArticleQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status='published')

    def by_author(self, author):
        return self.filter(author=author)

    def recent(self, days=30):
        cutoff = timezone.now() - timedelta(days=days)
        return self.filter(pub_date__gte=cutoff)

class Article(models.Model):
    objects = ArticleQuerySet.as_manager()

# Chainable: Article.objects.published().by_author(user).recent()
```


## Resources

- [Custom QuerySets](https://docs.djangoproject.com/en/6.0/topics/db/managers/#creating-a-manager-with-queryset-methods) — Creating managers with QuerySet methods

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*