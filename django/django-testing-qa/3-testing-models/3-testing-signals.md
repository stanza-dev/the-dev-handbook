---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-testing-signals"
---

# Testing Django Signals

## Introduction

Signals allow decoupled applications to respond to events. Testing them ensures side effects happen correctly.

## Key Concepts

**Signal**: Event notification system in Django.

**Receiver**: Function that responds to a signal.

## Deep Dive

### Testing Signal Receivers

```python
from django.db.models.signals import post_save
from unittest.mock import patch

class SignalTests(TestCase):
    def test_profile_created_on_user_save(self):
        user = User.objects.create_user('test', 'test@test.com', 'pass')
        self.assertTrue(Profile.objects.filter(user=user).exists())
    
    @patch('myapp.signals.send_welcome_email')
    def test_welcome_email_sent_on_registration(self, mock_send):
        User.objects.create_user('test', 'test@test.com', 'pass')
        mock_send.assert_called_once()
```

### Disabling Signals in Tests

```python
from django.db.models.signals import post_save
from myapp.models import Article
from myapp.signals import notify_subscribers

class TestWithoutSignals(TestCase):
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        post_save.disconnect(notify_subscribers, sender=Article)
    
    @classmethod
    def tearDownClass(cls):
        post_save.connect(notify_subscribers, sender=Article)
        super().tearDownClass()
    
    def test_article_without_notification(self):
        # Signal won't fire
        Article.objects.create(title='Test')
```

### Testing Custom Signals

```python
from myapp.signals import article_published

class CustomSignalTests(TestCase):
    def test_article_published_signal(self):
        received = []
        
        def handler(sender, article, **kwargs):
            received.append(article)
        
        article_published.connect(handler)
        try:
            article = Article.objects.create(title='Test')
            article.publish()  # Should emit signal
            
            self.assertEqual(len(received), 1)
            self.assertEqual(received[0], article)
        finally:
            article_published.disconnect(handler)
```

## Best Practices

1. **Mock external effects**: Email sending, API calls, etc.
2. **Disconnect when needed**: Some tests may need signals disabled.
3. **Always reconnect**: Use try/finally or setUp/tearDown.

## Summary

Test that signals trigger expected side effects. Use mocking for external services. Disconnect signals when testing in isolation and always reconnect them.

## Resources

- [Django Signals](https://docs.djangoproject.com/en/6.0/topics/signals/) — Django signals documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*