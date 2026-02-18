---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-assertions"
---

# Django Test Assertions

## Introduction

Django extends Python's unittest assertions with web-specific assertions for testing HTTP responses, templates, and forms.

## Key Concepts

**Assertion**: A statement that checks if a condition is true.

**AssertionError**: Raised when an assertion fails.

## Deep Dive

### Standard Assertions

```python
class BasicAssertionTests(TestCase):
    def test_assertions(self):
        # Equality
        self.assertEqual(1 + 1, 2)
        self.assertNotEqual(1, 2)
        
        # Boolean
        self.assertTrue(True)
        self.assertFalse(False)
        
        # None
        self.assertIsNone(None)
        self.assertIsNotNone('value')
        
        # Containment
        self.assertIn('a', ['a', 'b', 'c'])
        self.assertNotIn('d', ['a', 'b', 'c'])
        
        # Type checking
        self.assertIsInstance([], list)
```

### Django-Specific Assertions

```python
class DjangoAssertionTests(TestCase):
    def test_response_assertions(self):
        response = self.client.get('/articles/')
        
        # Template assertions
        self.assertTemplateUsed(response, 'articles/list.html')
        self.assertTemplateNotUsed(response, 'error.html')
        
        # Content assertions
        self.assertContains(response, 'Welcome')
        self.assertNotContains(response, 'Error')
        
        # Redirect assertions
        response = self.client.get('/old-url/')
        self.assertRedirects(response, '/new-url/')
```

### Exception Assertions

```python
from django.core.exceptions import ValidationError

class ExceptionTests(TestCase):
    def test_raises_exception(self):
        with self.assertRaises(ValidationError):
            raise ValidationError('Invalid')
        
        with self.assertRaises(ValueError) as cm:
            raise ValueError('Bad value')
        self.assertIn('Bad', str(cm.exception))
```

## Best Practices

1. **Use specific assertions**: `assertEqual` over `assertTrue(a == b)`.
2. **Check exception messages**: Use context manager to verify message.
3. **One assertion focus**: Each test should have one main assertion.

## Summary

Django provides web-specific assertions for templates, responses, and redirects. Use specific assertions for clearer error messages when tests fail.

## Resources

- [Test Assertions](https://docs.djangoproject.com/en/6.0/topics/testing/tools/#assertions) — Django test assertions reference

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*