---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-signals-intro"
---

# Introduction to Django Signals

Signals allow certain senders to notify a set of receivers when specific actions occur. They're useful for decoupling application components.

## Model Signals

Django provides these model-related signals:

| Signal | When Triggered |
|--------|----------------|
| `pre_save` | Before model.save() |
| `post_save` | After model.save() |
| `pre_delete` | Before model.delete() |
| `post_delete` | After model.delete() |
| `m2m_changed` | When ManyToMany changes |
| `pre_init` | Before __init__ |
| `post_init` | After __init__ |

## Basic Signal Usage

```python
from django.db.models.signals import post_save, pre_save, post_delete
from django.dispatch import receiver
from .models import Article, Profile


@receiver(post_save, sender=Article)
def article_saved(sender, instance, created, **kwargs):
    """
    Called after an Article is saved.
    
    Args:
        sender: The model class (Article)
        instance: The actual instance saved
        created: True if new record, False if update
        **kwargs: Additional arguments
    """
    if created:
        print(f"New article created: {instance.title}")
        # Send notification, update cache, etc.
    else:
        print(f"Article updated: {instance.title}")
```

## pre_save Signal

Modify the instance before saving:

```python
from django.db.models.signals import pre_save
from django.dispatch import receiver
from django.utils.text import slugify


@receiver(pre_save, sender=Article)
def auto_slug(sender, instance, **kwargs):
    """Auto-generate slug from title if not set."""
    if not instance.slug:
        instance.slug = slugify(instance.title)


@receiver(pre_save, sender=Article)
def set_word_count(sender, instance, **kwargs):
    """Calculate word count before saving."""
    if instance.body:
        instance.word_count = len(instance.body.split())
```

## post_save Signal

Perform actions after saving:

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User


@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    """Create a profile when a user is created."""
    if created:
        Profile.objects.create(user=instance)


@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    """Ensure profile is saved with user."""
    if hasattr(instance, 'profile'):
        instance.profile.save()
```

## Deletion Signals

```python
from django.db.models.signals import pre_delete, post_delete
from django.dispatch import receiver


@receiver(pre_delete, sender=Article)
def article_pre_delete(sender, instance, **kwargs):
    """Clean up before deletion."""
    # Delete associated files
    if instance.image:
        instance.image.delete(save=False)


@receiver(post_delete, sender=Article)
def article_post_delete(sender, instance, **kwargs):
    """Actions after deletion."""
    # Invalidate cache
    cache.delete(f'article:{instance.pk}')
    # Log the deletion
    logger.info(f"Article {instance.pk} deleted by signal")
```

## Connecting Signals

### Method 1: Decorator

```python
@receiver(post_save, sender=Article)
def my_handler(sender, instance, **kwargs):
    pass
```

### Method 2: Manual Connection

```python
from django.db.models.signals import post_save

def my_handler(sender, instance, **kwargs):
    pass

post_save.connect(my_handler, sender=Article)
```

## Where to Put Signal Handlers

Best practice: Create a `signals.py` file and import it in `apps.py`:

```python
# myapp/signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Article


@receiver(post_save, sender=Article)
def article_saved(sender, instance, created, **kwargs):
    if created:
        # Handle new article
        pass
```

```python
# myapp/apps.py
from django.apps import AppConfig


class MyappConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'

    def ready(self):
        # Import signals when app is ready
        import myapp.signals  # noqa
```

## Avoiding Infinite Loops

```python
@receiver(post_save, sender=Article)
def update_related(sender, instance, **kwargs):
    # BAD: This triggers another post_save!
    instance.updated_at = timezone.now()
    instance.save()  # Infinite loop!

# GOOD: Use update() which doesn't trigger signals
@receiver(post_save, sender=Article)
def update_related(sender, instance, **kwargs):
    Article.objects.filter(pk=instance.pk).update(
        updated_at=timezone.now()
    )
```

## Resources

- [Django Signals](https://docs.djangoproject.com/en/6.0/topics/signals/) — Official Django signals documentation

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*