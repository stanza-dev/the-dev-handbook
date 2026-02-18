---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-testing-model-properties"
---

# Testing Model Properties and Methods

## Introduction

Models often contain business logic in properties and methods. Testing these ensures your domain logic works correctly.

## Key Concepts

**Property**: Computed attribute using @property decorator.

**Model Method**: Instance or class method with business logic.

## Deep Dive

### Testing Properties

```python
class ArticlePropertyTests(TestCase):
    def test_word_count_property(self):
        article = Article(body='one two three four five')
        self.assertEqual(article.word_count, 5)
    
    def test_reading_time_property(self):
        # 200 words = 1 minute reading time
        article = Article(body=' '.join(['word'] * 400))
        self.assertEqual(article.reading_time, 2)
    
    def test_is_published_property(self):
        article = Article(status='published')
        self.assertTrue(article.is_published)
        
        draft = Article(status='draft')
        self.assertFalse(draft.is_published)
```

### Testing Instance Methods

```python
class ArticleMethodTests(TestCase):
    def test_publish_method(self):
        article = Article.objects.create(title='Test', status='draft')
        article.publish()
        
        self.assertEqual(article.status, 'published')
        self.assertIsNotNone(article.pub_date)
    
    def test_archive_method(self):
        article = Article.objects.create(title='Test', status='published')
        article.archive()
        
        article.refresh_from_db()
        self.assertEqual(article.status, 'archived')
```

### Testing Class Methods

```python
class ArticleClassMethodTests(TestCase):
    def test_published_count(self):
        Article.objects.create(title='A', status='published')
        Article.objects.create(title='B', status='published')
        Article.objects.create(title='C', status='draft')
        
        self.assertEqual(Article.published_count(), 2)
```

## Best Practices

1. **Test edge cases**: Empty strings, None values, boundary conditions.
2. **Use refresh_from_db()**: After operations that modify the database.
3. **Test return values**: Verify methods return expected values.

## Summary

Test model properties and methods as units of business logic. Verify both happy paths and edge cases. Use refresh_from_db() to ensure database state is current.

## Resources

- [Model Methods](https://docs.djangoproject.com/en/6.0/topics/db/models/#model-methods) — Django model methods guide

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*