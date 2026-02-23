---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-test-basics"
---

# Introduction to Django Testing

## Introduction

Django ships with a robust testing framework built on top of Python's unittest module. Understanding how to write and organize tests is fundamental to building reliable Django applications. This lesson covers the core testing infrastructure, test structure, and essential assertions.

## Key Concepts

**TestCase**: Django's base test class that provides database isolation via transactions.

**Test Runner**: The command `python manage.py test` that discovers and runs tests.

**Assertion**: A method that checks whether a condition is true and fails the test if not.

**Test Database**: A separate database Django creates automatically for test isolation.

## Real World Context

In production Django projects, tests are the safety net that lets teams deploy confidently. Without tests, every code change risks breaking existing features. Companies like Instagram and Disqus run thousands of Django tests on every pull request to catch regressions before they reach users.

## Deep Dive

### Test Structure

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

### Running Tests

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

### Test Organization

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

### TestCase Classes

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

### Assertions

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

## Common Pitfalls

1. **Forgetting to run migrations before tests**: Tests use the current schema, so always run `makemigrations` and `migrate` first.
2. **Tests depending on execution order**: Each test should be independent. Never rely on state from a previous test.
3. **Using `print()` for debugging instead of `--verbosity=2`**: The test runner's verbose mode gives structured output that is easier to parse.

## Best Practices

1. **One test file per module**: Mirror your app structure in your test directory.
2. **Use `--parallel` in CI**: Run tests in parallel to speed up your test suite.
3. **Choose the right TestCase class**: Use SimpleTestCase when you do not need the database.

## Summary

- Django's test framework extends Python's unittest with web-specific features
- Use `python manage.py test` to discover and run tests
- Choose between SimpleTestCase, TestCase, and TransactionTestCase based on needs
- Organize tests in a dedicated `tests/` directory mirroring your app structure
- Use assertions to verify expected behavior in each test method

## Code Examples

**A basic Django TestCase that creates an article in setUp and tests creation and string representation.**

```python
from django.test import TestCase
from .models import Article


class ArticleModelTests(TestCase):
    def setUp(self):
        self.article = Article.objects.create(
            title='Test Article',
            body='Test content',
            status='draft'
        )

    def test_article_creation(self):
        self.assertEqual(self.article.title, 'Test Article')
        self.assertEqual(self.article.status, 'draft')
        self.assertIsNotNone(self.article.pk)

    def test_article_str(self):
        self.assertEqual(str(self.article), 'Test Article')

```


## Resources

- [Testing in Django](https://docs.djangoproject.com/en/6.0/topics/testing/) — Official testing documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*