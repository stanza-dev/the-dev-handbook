---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-test-basics"
---

# Introduction to Django Testing

Django includes a testing framework based on Python's unittest module, with additional features for web development.

## Why Test?

Tests ensure your code works correctly and continues to work as you make changes:
- **Catch bugs early** before they reach production
- **Refactor with confidence** knowing tests will catch regressions
- **Document behavior** through test cases
- **Speed up development** by reducing manual testing

## Test Structure

```python
# tests.py or tests/test_models.py
from django.test import TestCase
from .models import Article


class ArticleModelTests(TestCase):
    """Tests for the Article model."""
    
    def setUp(self):
        """Set up test data before each test method."""
        self.article = Article.objects.create(
            title='Test Article',
            body='This is a test article.',
            status='draft'
        )
    
    def test_article_creation(self):
        """Test that an article can be created."""
        self.assertEqual(self.article.title, 'Test Article')
        self.assertEqual(self.article.status, 'draft')
    
    def test_article_str(self):
        """Test the string representation."""
        self.assertEqual(str(self.article), 'Test Article')
    
    def test_article_publish(self):
        """Test publishing an article."""
        self.article.publish()
        self.assertEqual(self.article.status, 'published')
        self.assertIsNotNone(self.article.pub_date)
```

## Running Tests

```bash
# Run all tests
python manage.py test

# Run tests for a specific app
python manage.py test blog

# Run a specific test class
python manage.py test blog.tests.ArticleModelTests

# Run a specific test method
python manage.py test blog.tests.ArticleModelTests.test_article_creation

# Run with verbosity
python manage.py test --verbosity=2

# Run in parallel
python manage.py test --parallel

# Keep test database for debugging
python manage.py test --keepdb
```

## Test Organization

```
blog/
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_views.py
│   ├── test_forms.py
│   └── test_api.py
├── models.py
├── views.py
└── forms.py
```

## TestCase Classes

```python
from django.test import TestCase, SimpleTestCase, TransactionTestCase


class SimpleTests(SimpleTestCase):
    """For tests that don't need database access."""
    
    def test_utility_function(self):
        from .utils import slugify
        self.assertEqual(slugify('Hello World'), 'hello-world')


class ModelTests(TestCase):
    """For tests with database access (uses transactions)."""
    
    def test_model_save(self):
        article = Article.objects.create(title='Test')
        self.assertTrue(Article.objects.filter(pk=article.pk).exists())


class TransactionTests(TransactionTestCase):
    """For tests that need to test transaction behavior."""
    
    def test_atomic_rollback(self):
        from django.db import transaction
        
        try:
            with transaction.atomic():
                Article.objects.create(title='Test')
                raise Exception('Rollback!')
        except Exception:
            pass
        
        self.assertEqual(Article.objects.count(), 0)
```

## Assertions

```python
class ArticleTests(TestCase):
    def test_assertions(self):
        article = Article.objects.create(title='Test', status='draft')
        
        # Equality
        self.assertEqual(article.status, 'draft')
        self.assertNotEqual(article.status, 'published')
        
        # Boolean
        self.assertTrue(article.is_draft)
        self.assertFalse(article.is_published)
        
        # None
        self.assertIsNone(article.pub_date)
        self.assertIsNotNone(article.created_at)
        
        # Containment
        self.assertIn('Test', article.title)
        self.assertNotIn('hello', article.title)
        
        # Instance
        self.assertIsInstance(article, Article)
        
        # Comparison
        self.assertGreater(len(article.title), 0)
        self.assertLess(article.views, 100)
        
        # Exception
        with self.assertRaises(ValidationError):
            article.full_clean()  # Missing required field
```

## Resources

- [Testing in Django](https://docs.djangoproject.com/en/6.0/topics/testing/) — Official testing documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*