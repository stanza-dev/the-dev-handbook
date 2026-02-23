---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-m2m-signals"
---

# ManyToMany Signals

The `m2m_changed` signal is triggered when ManyToMany relationships are modified.

## m2m_changed Signal

```python
from django.db.models.signals import m2m_changed
from django.dispatch import receiver


@receiver(m2m_changed, sender=Article.tags.through)
def tags_changed(sender, instance, action, pk_set, **kwargs):
    """
    Called when Article tags change.
    
    Args:
        sender: The intermediate model (Article_tags)
        instance: The Article instance
        action: Type of change (see below)
        pk_set: Set of primary keys being added/removed
    """
    print(f"Action: {action}")
    print(f"PKs: {pk_set}")
```

## Action Types

| Action | Description |
|--------|-------------|
| `pre_add` | Before adding items |
| `post_add` | After adding items |
| `pre_remove` | Before removing items |
| `post_remove` | After removing items |
| `pre_clear` | Before clearing all items |
| `post_clear` | After clearing all items |

## Practical Example: Tag Counter

```python
from django.db.models.signals import m2m_changed
from django.dispatch import receiver


@receiver(m2m_changed, sender=Article.tags.through)
def update_tag_counts(sender, instance, action, pk_set, **kwargs):
    """Update article count on tags when relationships change."""
    
    if action == 'post_add':
        # Increment count for added tags
        Tag.objects.filter(pk__in=pk_set).update(
            article_count=F('article_count') + 1
        )
    
    elif action == 'post_remove':
        # Decrement count for removed tags
        Tag.objects.filter(pk__in=pk_set).update(
            article_count=F('article_count') - 1
        )
    
    elif action == 'post_clear':
        # Recalculate all tag counts (expensive but accurate)
        for tag in Tag.objects.all():
            tag.article_count = tag.articles.count()
            tag.save()
```

## Handling Both Directions

```python
@receiver(m2m_changed, sender=Article.tags.through)
def tags_changed(sender, instance, action, reverse, pk_set, **kwargs):
    """
    Handle changes from either direction.
    
    reverse=False: article.tags.add(tag)
    reverse=True:  tag.articles.add(article)
    """
    if reverse:
        # instance is a Tag, pk_set contains Article PKs
        articles = Article.objects.filter(pk__in=pk_set)
    else:
        # instance is an Article, pk_set contains Tag PKs
        tags = Tag.objects.filter(pk__in=pk_set)
```

## Custom Signals

Create your own signals for application-specific events:

```python
# signals.py
from django.dispatch import Signal

# Define custom signals
article_published = Signal()  # Sent when article is published
order_completed = Signal()    # Sent when order is completed
user_upgraded = Signal()      # Sent when user upgrades account
```

```python
# views.py or models.py
from .signals import article_published


def publish_article(article):
    article.status = 'published'
    article.pub_date = timezone.now()
    article.save()
    
    # Send custom signal
    article_published.send(
        sender=Article,
        article=article,
        published_by=request.user
    )
```

```python
# handlers.py
from django.dispatch import receiver
from .signals import article_published


@receiver(article_published)
def notify_subscribers(sender, article, published_by, **kwargs):
    """Send notifications when article is published."""
    subscribers = article.author.subscribers.all()
    for subscriber in subscribers:
        send_notification(
            subscriber,
            f"New article: {article.title}"
        )


@receiver(article_published)
def update_search_index(sender, article, **kwargs):
    """Update search index when article is published."""
    search_client.index_document('articles', article.to_dict())
```

## When to Use Signals vs Model Methods

**Use Signals When:**
- Decoupling is important
- Multiple handlers from different apps
- Third-party app integration
- Cross-cutting concerns (logging, caching)

**Use Model Methods When:**
- Logic is tightly coupled to the model
- You need to control execution order
- Simpler, more explicit code flow
- Performance is critical (signals have overhead)

```python
# Model method approach (alternative to signals)
class Article(models.Model):
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)
```\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test manytomany signals with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some manytomany signals features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- ManyToMany Signals is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**m2m_changed signal for reacting to ManyToMany relationship changes**

```python
from django.db.models.signals import m2m_changed
from django.dispatch import receiver

@receiver(m2m_changed, sender=Article.tags.through)
def handle_tags_changed(sender, instance, action, pk_set, **kwargs):
    if action == 'post_add':
        instance.update_search_index()
    elif action == 'post_remove':
        instance.update_search_index()
    elif action == 'post_clear':
        instance.clear_search_index()
```


## Resources

- [m2m_changed Signal](https://docs.djangoproject.com/en/6.0/ref/signals/#m2m-changed) — ManyToMany changed signal reference

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*