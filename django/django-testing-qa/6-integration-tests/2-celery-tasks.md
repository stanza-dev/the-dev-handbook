---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-testing-celery-tasks"
---

# Testing Celery Tasks

## Introduction

Celery tasks run asynchronously and need special testing approaches to verify both task logic and integration.

## Key Concepts

**Task Unit Test**: Test task function directly without Celery.

**Eager Mode**: Execute tasks synchronously for testing.

## Deep Dive

### Testing Task Logic Directly

```python
class TaskLogicTests(TestCase):
    def test_send_notification_task(self):
        user = User.objects.create_user('test', 'test@test.com', 'pass')
        # Call task function directly (not .delay())
        result = send_notification(user.id, 'Test message')
        
        self.assertTrue(result['success'])
        self.assertEqual(Notification.objects.count(), 1)
```

### Using task_always_eager

```python
# settings/test.py (Celery 5+ lowercase setting names)
task_always_eager = True
task_eager_propagates = True

# In tests - .delay() now runs synchronously
class EagerTaskTests(TestCase):
    def test_task_runs_immediately(self):
        result = process_order.delay(order_id=1)
        self.assertTrue(result.successful())
```

### Mocking Task Calls

```python
from unittest.mock import patch

class ViewWithTaskTests(TestCase):
    @patch('myapp.views.send_welcome_email.delay')
    def test_registration_queues_email(self, mock_task):
        response = self.client.post(reverse('register'), {
            'email': 'new@test.com',
            'password': 'testpass123',
        })
        
        mock_task.assert_called_once()
        args = mock_task.call_args[0]
        self.assertEqual(args[0], 'new@test.com')
```

### Testing Retries

```python
from celery.exceptions import Retry

class RetryTests(TestCase):
    @patch('myapp.tasks.external_api_call')
    def test_task_retries_on_failure(self, mock_api):
        mock_api.side_effect = ConnectionError()
        
        with self.assertRaises(Retry):
            sync_data()
```

## Real World Context

Celery tasks handle background work in Django applications: sending emails, processing uploads, generating reports, and syncing data with external systems. Testing them is tricky because they normally run in a separate process. The three approaches (direct call, eager mode, mocking) each serve different testing needs.

**Note:** Django 6.0 introduced a built-in background tasks framework (`django.tasks`) that provides a lighter-weight alternative to Celery for many use cases. The testing principles covered here — direct function calls, eager execution, and mocking — apply equally to Django's built-in tasks.

## Common Pitfalls

1. **Testing with a real Celery worker**: Tests should never depend on a running Celery broker. Use eager mode or direct calls.
2. **Forgetting to test retry logic**: If your task uses `self.retry()`, test that retries happen on expected exceptions.
3. **Not testing task arguments**: When views call `task.delay(user_id)`, verify the correct arguments are passed using mock assertions.

## Best Practices

1. **Use ALWAYS_EAGER for integration tests**: Runs tasks synchronously.
2. **Test task logic separately**: Call function directly without .delay().
3. **Mock for unit tests**: Verify .delay() was called correctly.

## Summary

Test Celery task logic by calling functions directly. Use task_always_eager for integration tests. Mock .delay() calls when testing views that queue tasks.

## Resources

- [Testing Celery](https://docs.celeryq.dev/en/stable/userguide/testing.html) — Celery testing documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*