---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-custom-signals"
---

# Custom Signals

## Introduction

Beyond built-in model signals, Django lets you define your own signals for application-specific events. Custom signals decouple components and enable extensible architectures.

## Key Concepts

- **Signal**: A `Signal()` instance that acts as a notification channel.
- **Sender**: The class or object that sends the signal.
- **Receiver**: A function connected to the signal that reacts when it fires.

## Real World Context

Custom signals are used for publishing events (article published, payment received), cross-module side effects (update search index), and plugin systems where third-party code hooks into your application.

## Deep Dive

### Defining and Sending

To fire a signal, call `.send()` with the sender class and any keyword arguments your receivers expect:

```python
# myapp/signals.py
from django.dispatch import Signal

article_published = Signal()
order_completed = Signal()
```

```python
# myapp/services.py
from .signals import article_published

def publish_article(article, user):
    article.status = 'published'
    article.save()
    article_published.send(sender=article.__class__, article=article, user=user)
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Receiving

Connect receiver functions to signals using the `@receiver` decorator. Each receiver runs when the signal is sent:

```python
from django.dispatch import receiver
from myapp.signals import article_published

@receiver(article_published)
def notify_subscribers(sender, article, user, **kwargs):
    for sub in article.author.subscribers.all():
        send_email(sub.email, f'New: {article.title}')
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### send() vs send_robust()

Django offers two ways to send signals: `send()` propagates exceptions from receivers, while `send_robust()` catches them and returns results:

```python
# send() raises if any receiver raises
article_published.send(sender=Article, article=article)

# send_robust() catches exceptions
results = article_published.send_robust(sender=Article, article=article)
for receiver, response in results:
    if isinstance(response, Exception):
        logger.error(f'Handler failed: {response}')
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Register in AppConfig

Signal handlers must be imported at startup for Django to register them. The standard pattern is importing your handlers module in `AppConfig.ready()`:

```python
class NotificationsConfig(AppConfig):
    name = 'notifications'

    def ready(self):
        import notifications.handlers  # noqa: F401
```

## Common Pitfalls

1. **Forgetting `**kwargs`** — Receivers must accept `**kwargs` for forward compatibility.
2. **Side effects without `on_commit`** — Receivers inside transactions should use `transaction.on_commit` for email/HTTP calls.
3. **Duplicate connections** — Use `dispatch_uid` to prevent the same handler being connected twice.

## Best Practices

1. **Use `send_robust()` in production** — Prevents one failing handler from breaking the chain.
2. **Keep handlers lightweight** — Defer heavy work to background tasks.
3. **Register in `AppConfig.ready()`** — Ensures single registration.

## Summary

- Custom signals are defined with `Signal()` and sent with `.send()` or `.send_robust()`.
- Connect receivers with `@receiver` or `.connect(dispatch_uid=...)`.
- Register handlers in `AppConfig.ready()`.
- Use `send_robust()` in production for fault tolerance.

## Code Examples

**Defining, sending, and receiving custom signals**

```python
from django.dispatch import Signal, receiver

article_published = Signal()

@receiver(article_published)
def notify(sender, article, **kwargs):
    send_email(article.author.email, f'Published: {article.title}')

# Send:
article_published.send(sender=Article, article=article)
```


## Resources

- [Defining and Sending Signals](https://docs.djangoproject.com/en/6.0/topics/signals/#defining-and-sending-signals) — Official guide to creating custom signals

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*