---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-mocking-external-services"
---

# Mocking External Services

## Introduction

External services (APIs, databases, email) should be mocked in tests for speed, reliability, and isolation.

## Key Concepts

**Mock**: Fake object that replaces a real dependency.

**Responses Library**: Mock HTTP requests elegantly.

## Deep Dive

### Mocking HTTP Requests

```python
import responses
import requests

class ExternalAPITests(TestCase):
    @responses.activate
    def test_fetch_weather_data(self):
        responses.add(
            responses.GET,
            'https://api.weather.com/current',
            json={'temp': 72, 'condition': 'sunny'},
            status=200
        )
        
        result = get_weather('NYC')
        self.assertEqual(result['temp'], 72)
    
    @responses.activate
    def test_api_error_handling(self):
        responses.add(
            responses.GET,
            'https://api.weather.com/current',
            json={'error': 'Service unavailable'},
            status=503
        )
        
        with self.assertRaises(ServiceUnavailable):
            get_weather('NYC')
```

### Mocking with unittest.mock

```python
from unittest.mock import patch, Mock

class PaymentTests(TestCase):
    @patch('myapp.payments.stripe.Charge.create')
    def test_successful_payment(self, mock_charge):
        mock_charge.return_value = Mock(id='ch_123', status='succeeded')
        
        result = process_payment(amount=100, token='tok_visa')
        
        self.assertTrue(result.success)
        mock_charge.assert_called_once_with(
            amount=10000,  # cents
            currency='usd',
            source='tok_visa'
        )
```

### Mocking Celery Tasks

```python
from unittest.mock import patch

class TaskTests(TestCase):
    @patch('myapp.tasks.send_notification.delay')
    def test_notification_queued(self, mock_delay):
        create_order(user=self.user)
        mock_delay.assert_called_once()
```

## Real World Context

Modern Django applications integrate with payment processors (Stripe, PayPal), email services (SendGrid, SES), cloud storage (S3), and dozens of third-party APIs. Without mocking, your test suite would require active accounts with every service, internet access, and would run orders of magnitude slower. Mocking is not optional for professional Django development.

## Common Pitfalls

1. **Mocking too deep**: Mock at the boundary (requests.get, stripe.Charge.create), not internal helper functions.
2. **Forgetting to test error responses**: Mock both success and failure cases -- 200 and 503 responses from external APIs.
3. **Not verifying request parameters**: Use assert_called_with() to ensure your code sends the right data to external services.

## Best Practices

1. **Mock at the boundary**: Mock where your code meets external systems.
2. **Use responses for HTTP**: Cleaner than patching requests.
3. **Verify calls**: Use assert_called_with() to verify correct usage.

## Summary

Mock external services to make tests fast and reliable. Use the responses library for HTTP mocking. Always verify mocks were called correctly.

## Resources

- [unittest.mock](https://docs.python.org/3/library/unittest.mock.html) — Python mock documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*