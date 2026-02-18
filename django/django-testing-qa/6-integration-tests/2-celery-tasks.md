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

### Using CELERY_TASK_ALWAYS_EAGER

```python
# settings/test.py
CELERY_TASK_ALWAYS_EAGER = True
CELERY_TASK_EAGER_PROPAGATES = True

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

## Best Practices

1. **Use ALWAYS_EAGER for integration tests**: Runs tasks synchronously.
2. **Test task logic separately**: Call function directly without .delay().
3. **Mock for unit tests**: Verify .delay() was called correctly.

## Summary

Test Celery task logic by calling functions directly. Use CELERY_TASK_ALWAYS_EAGER for integration tests. Mock .delay() calls when testing views that queue tasks.

## Resources

- [Testing Celery](https://docs.celeryq.dev/en/stable/userguide/testing.html) — Celery testing documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*