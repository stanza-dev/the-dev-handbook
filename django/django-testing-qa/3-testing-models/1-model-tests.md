---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-model-tests"
---

# Model Testing Strategies

## Introduction

Model tests verify that your data layer works correctly, including creation, validation, relationships, custom methods, and query managers. Since models are the foundation of a Django application, thorough model testing prevents data integrity issues from propagating through the entire system.

## Key Concepts

**full_clean()**: Triggers all model validation including field validators and the custom clean() method.

**IntegrityError**: Database-level exception raised when constraints like unique or foreign key are violated.

**refresh_from_db()**: Reloads an object's field values from the database after external modifications.

## Real World Context

Model tests are fast because they interact directly with the database without HTTP overhead. In production codebases, model tests often make up 40-60% of the test suite because they cover business logic, validation rules, and data relationships. A bug in a model method can cascade through views, serializers, and templates, so catching it early at the model level saves significant debugging time.

## Deep Dive

## Testing Model Creation

```python
from django.test import TestCase
from django.core.exceptions import ValidationError
from django.db import IntegrityError
from .models import Article, Category


class ArticleModelTests(TestCase):
    def test_create_article(self):
        """Test basic article creation."""
        article = Article.objects.create(
            title='Test Article',
            body='Test content',
            status='draft'
        )
        
        self.assertEqual(article.title, 'Test Article')
        self.assertEqual(article.status, 'draft')
        self.assertIsNotNone(article.pk)
        self.assertIsNotNone(article.created_at)
    
    def test_auto_slug_generation(self):
        """Test that slug is auto-generated from title."""
        article = Article.objects.create(
            title='Hello World Article'
        )
        
        self.assertEqual(article.slug, 'hello-world-article')
    
    def test_unique_slug(self):
        """Test that duplicate slugs are handled."""
        Article.objects.create(title='Test')
        article2 = Article.objects.create(title='Test')
        
        self.assertNotEqual(article2.slug, 'test')  # Should be unique
```

## Testing Model Methods

```python
class ArticleMethodTests(TestCase):
    def setUp(self):
        self.article = Article.objects.create(
            title='Test',
            body='Test content',
            status='draft'
        )
    
    def test_publish_method(self):
        """Test the publish() method."""
        self.article.publish()
        
        self.assertEqual(self.article.status, 'published')
        self.assertIsNotNone(self.article.pub_date)
    
    def test_unpublish_method(self):
        """Test the unpublish() method."""
        self.article.publish()
        self.article.unpublish()
        
        self.assertEqual(self.article.status, 'draft')
        self.assertIsNone(self.article.pub_date)
    
    def test_word_count_property(self):
        """Test the word_count property."""
        self.article.body = 'One two three four five'
        self.assertEqual(self.article.word_count, 5)
    
    def test_reading_time(self):
        """Test reading time calculation."""
        self.article.body = ' '.join(['word'] * 400)  # 400 words
        self.assertEqual(self.article.reading_time, 2)  # ~200 words/min
```

## Testing Validation

```python
class ArticleValidationTests(TestCase):
    def test_title_required(self):
        """Test that title is required."""
        article = Article(body='Content', status='draft')
        
        with self.assertRaises(ValidationError) as cm:
            article.full_clean()
        
        self.assertIn('title', cm.exception.message_dict)
    
    def test_title_max_length(self):
        """Test title max length validation."""
        article = Article(
            title='x' * 201,  # Over 200 char limit
            body='Content'
        )
        
        with self.assertRaises(ValidationError) as cm:
            article.full_clean()
        
        self.assertIn('title', cm.exception.message_dict)
    
    def test_status_choices(self):
        """Test that status must be a valid choice."""
        article = Article(title='Test', status='invalid')
        
        with self.assertRaises(ValidationError):
            article.full_clean()
    
    def test_custom_validation(self):
        """Test custom clean method."""
        article = Article(
            title='Test',
            status='published',
            pub_date=None  # Published articles need pub_date
        )
        
        with self.assertRaises(ValidationError) as cm:
            article.full_clean()
        
        self.assertIn('Published articles must have a publication date', str(cm.exception))
```

## Testing Relationships

```python
class ArticleRelationshipTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        cls.author = User.objects.create_user('author', 'a@test.com', 'pass')
        cls.category = Category.objects.create(name='Tech')
    
    def test_author_relationship(self):
        """Test ForeignKey to User."""
        article = Article.objects.create(
            title='Test',
            author=self.author
        )
        
        self.assertEqual(article.author, self.author)
        self.assertIn(article, self.author.articles.all())
    
    def test_category_relationship(self):
        """Test ManyToMany relationship."""
        article = Article.objects.create(title='Test')
        article.categories.add(self.category)
        
        self.assertIn(self.category, article.categories.all())
        self.assertIn(article, self.category.articles.all())
    
    def test_cascade_delete(self):
        """Test that deleting author deletes articles."""
        article = Article.objects.create(
            title='Test',
            author=self.author
        )
        article_pk = article.pk
        
        self.author.delete()
        
        self.assertFalse(Article.objects.filter(pk=article_pk).exists())
```

## Testing Querysets and Managers

```python
class ArticleManagerTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        cls.published1 = Article.objects.create(
            title='Published 1',
            status='published',
            pub_date=timezone.now()
        )
        cls.published2 = Article.objects.create(
            title='Published 2',
            status='published',
            pub_date=timezone.now()
        )
        cls.draft = Article.objects.create(
            title='Draft',
            status='draft'
        )
    
    def test_published_manager(self):
        """Test the published() manager method."""
        published = Article.objects.published()
        
        self.assertEqual(published.count(), 2)
        self.assertIn(self.published1, published)
        self.assertNotIn(self.draft, published)
    
    def test_drafts_manager(self):
        """Test the drafts() manager method."""
        drafts = Article.objects.drafts()
        
        self.assertEqual(drafts.count(), 1)
        self.assertIn(self.draft, drafts)
```

## Common Pitfalls

1. **Forgetting that save() does not call full_clean()**: Django's save() skips validation by default. Always call full_clean() explicitly in tests to verify validators.
2. **Not using refresh_from_db() after database operations**: When another query or method modifies the database, your in-memory object becomes stale.
3. **Testing Django internals**: Do not test that Django's ORM creates objects correctly. Test your custom logic, methods, and validation rules.

## Best Practices

1. **Test validation explicitly**: Call full_clean() and verify expected ValidationError messages.
2. **Test edge cases**: Empty strings, None values, boundary lengths, and duplicate values.
3. **Use setUpTestData for shared read-only data**: Related objects like users and categories rarely change between tests.

## Summary

- Test model creation, validation, methods, relationships, and managers
- Always call full_clean() to trigger validation in tests
- Use refresh_from_db() after database modifications
- Test custom managers and querysets for filtering logic
- Focus on testing your custom business logic, not Django internals

## Code Examples

**Testing model validation with full_clean() and a model method with refresh_from_db().**

```python
from django.test import TestCase
from django.core.exceptions import ValidationError
from .models import Article


class ArticleValidationTests(TestCase):
    def test_title_required(self):
        article = Article(body='Content', status='draft')
        with self.assertRaises(ValidationError) as cm:
            article.full_clean()
        self.assertIn('title', cm.exception.message_dict)

    def test_publish_sets_date(self):
        article = Article.objects.create(
            title='Test', status='draft'
        )
        article.publish()
        article.refresh_from_db()
        self.assertEqual(article.status, 'published')
        self.assertIsNotNone(article.pub_date)

```


## Resources

- [Model Testing](https://docs.djangoproject.com/en/6.0/topics/testing/overview/#writing-tests) — Writing model tests

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*