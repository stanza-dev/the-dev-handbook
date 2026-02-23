---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-test-fixtures"
---

# Test Data and Fixtures

## Introduction

Managing test data efficiently is one of the most important skills for writing maintainable tests. Poor test data management leads to slow, brittle tests that are hard to understand. This lesson covers setUp methods, JSON fixtures, and the factory pattern.

## Key Concepts

**setUpTestData**: Class method that creates shared test data once per TestCase class (fast, read-only).

**setUp**: Instance method that creates fresh test data before each test method.

**Fixture**: A JSON or YAML file containing serialized database records for loading into tests.

**Factory**: A pattern that creates test objects with sensible defaults and easy customization.

## Real World Context

Large Django projects can have thousands of tests. If each test creates its own data from scratch via setUp(), the test suite becomes painfully slow. Factories and setUpTestData are how teams at scale keep test suites running in minutes instead of hours. Libraries like factory_boy are standard in most professional Django codebases.

## Deep Dive

## setUp and setUpTestData

```python
from django.test import TestCase
from django.contrib.auth.models import User
from .models import Article, Category


class ArticleTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        """
        Set up data for the whole TestCase.
        Called once for the entire class (faster).
        Data is shared and should not be modified.
        """
        cls.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        cls.category = Category.objects.create(name='Technology')
    
    def setUp(self):
        """
        Set up data for each test method.
        Called before every test method.
        Use for data that might be modified.
        """
        self.article = Article.objects.create(
            title='Test Article',
            body='Test body',
            author=self.user,
            category=self.category
        )
    
    def test_article_author(self):
        self.assertEqual(self.article.author, self.user)
    
    def test_article_modification(self):
        # This modifies the article, but setUp creates a fresh one for each test
        self.article.title = 'Modified Title'
        self.article.save()
        self.assertEqual(self.article.title, 'Modified Title')
```

## JSON Fixtures

```json
// blog/fixtures/test_data.json
[
    {
        "model": "auth.user",
        "pk": 1,
        "fields": {
            "username": "testuser",
            "email": "test@example.com",
            "password": "pbkdf2_sha256$..."
        }
    },
    {
        "model": "blog.category",
        "pk": 1,
        "fields": {
            "name": "Technology",
            "slug": "technology"
        }
    },
    {
        "model": "blog.article",
        "pk": 1,
        "fields": {
            "title": "Test Article",
            "slug": "test-article",
            "body": "Test content",
            "author": 1,
            "category": 1,
            "status": "published"
        }
    }
]
```

```python
class ArticleWithFixtureTests(TestCase):
    fixtures = ['test_data.json']
    
    def test_fixture_loaded(self):
        self.assertEqual(Article.objects.count(), 1)
        article = Article.objects.get(pk=1)
        self.assertEqual(article.title, 'Test Article')
```

## Factory Pattern

```python
# tests/factories.py
from django.contrib.auth.models import User
from blog.models import Article, Category


class UserFactory:
    """Factory for creating test users."""
    counter = 0
    
    @classmethod
    def create(cls, **kwargs):
        cls.counter += 1
        defaults = {
            'username': f'user{cls.counter}',
            'email': f'user{cls.counter}@example.com',
            'password': 'testpass123',
        }
        defaults.update(kwargs)
        password = defaults.pop('password')
        user = User.objects.create_user(**defaults)
        user.raw_password = password  # Store for test reference
        return user


class ArticleFactory:
    """Factory for creating test articles."""
    counter = 0
    
    @classmethod
    def create(cls, **kwargs):
        cls.counter += 1
        if 'author' not in kwargs:
            kwargs['author'] = UserFactory.create()
        
        defaults = {
            'title': f'Test Article {cls.counter}',
            'body': 'Test content',
            'status': 'draft',
        }
        defaults.update(kwargs)
        return Article.objects.create(**defaults)
    
    @classmethod
    def create_published(cls, **kwargs):
        kwargs['status'] = 'published'
        from django.utils import timezone
        kwargs['pub_date'] = timezone.now()
        return cls.create(**kwargs)
```

```python
# Using factories in tests
from .factories import ArticleFactory, UserFactory


class ArticleTests(TestCase):
    def test_with_factory(self):
        article = ArticleFactory.create(title='Custom Title')
        self.assertEqual(article.title, 'Custom Title')
    
    def test_multiple_articles(self):
        articles = [ArticleFactory.create() for _ in range(5)]
        self.assertEqual(Article.objects.count(), 5)
    
    def test_published_article(self):
        article = ArticleFactory.create_published()
        self.assertEqual(article.status, 'published')
        self.assertIsNotNone(article.pub_date)
```

## Using factory_boy

```python
# pip install factory_boy
import factory
from factory.django import DjangoModelFactory
from django.contrib.auth.models import User
from blog.models import Article


class UserFactory(DjangoModelFactory):
    class Meta:
        model = User
    
    username = factory.Sequence(lambda n: f'user{n}')
    email = factory.LazyAttribute(lambda o: f'{o.username}@example.com')
    password = factory.PostGenerationMethodCall('set_password', 'testpass123')


class ArticleFactory(DjangoModelFactory):
    class Meta:
        model = Article
    
    title = factory.Sequence(lambda n: f'Article {n}')
    body = factory.Faker('paragraph')
    author = factory.SubFactory(UserFactory)
    status = 'draft'
    
    class Params:
        published = factory.Trait(
            status='published',
            pub_date=factory.LazyFunction(timezone.now)
        )
```

## Common Pitfalls

1. **Modifying setUpTestData objects**: Data created by setUpTestData is shared across all tests in the class. Modifying it corrupts other tests.
2. **Overusing JSON fixtures**: Fixtures are hard to maintain as your schema evolves. Prefer factories for complex data.
3. **Creating unnecessary data**: Each test should create only the data it needs. Excessive setup slows tests and obscures intent.

## Best Practices

1. **Use setUpTestData for read-only data**: It runs once per class, making tests significantly faster.
2. **Use factories over fixtures**: Factories are more flexible and self-documenting.
3. **Keep test data minimal**: Create only what each test requires.

## Summary

- setUpTestData creates data once per class (fast); setUp creates per test (fresh)
- JSON fixtures load pre-defined data but are hard to maintain
- The factory pattern creates test data with sensible defaults
- factory_boy is the industry-standard library for Django test factories
- Choose the right approach based on whether tests need shared or isolated data

## Resources

- [Test Fixtures](https://docs.djangoproject.com/en/6.0/topics/testing/tools/#fixture-loading) — Loading fixtures in tests

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*