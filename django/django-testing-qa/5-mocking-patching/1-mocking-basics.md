---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-mocking-basics"
---

# Introduction to Mocking

Mocking replaces real objects with fake ones to isolate tests from external dependencies.

## Why Mock?

- **Speed**: Avoid slow operations (network, filesystem)
- **Isolation**: Test units independently
- **Reliability**: Don't depend on external services
- **Control**: Simulate error conditions

## Basic Mocking

```python
from unittest.mock import Mock, patch, MagicMock
from django.test import TestCase


class EmailTests(TestCase):
    @patch('blog.views.send_mail')
    def test_article_publish_sends_email(self, mock_send_mail):
        """Test that publishing sends notification email."""
        article = Article.objects.create(
            title='Test',
            author=self.user
        )
        
        # Call the view or method that sends email
        article.publish()
        
        # Assert send_mail was called
        mock_send_mail.assert_called_once()
        
        # Check call arguments
        call_args = mock_send_mail.call_args
        self.assertIn('Test', call_args[1]['subject'])
        self.assertEqual(call_args[1]['recipient_list'], [self.user.email])
```

## Patching Methods

```python
class ExternalAPITests(TestCase):
    @patch('blog.services.requests.get')
    def test_fetch_external_data(self, mock_get):
        """Test fetching data from external API."""
        # Configure mock response
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = {
            'articles': [{'title': 'External Article'}]
        }
        mock_get.return_value = mock_response
        
        # Call the function that uses requests
        from blog.services import fetch_external_articles
        result = fetch_external_articles()
        
        # Verify
        mock_get.assert_called_once_with(
            'https://api.example.com/articles',
            timeout=10
        )
        self.assertEqual(len(result), 1)
        self.assertEqual(result[0]['title'], 'External Article')
    
    @patch('blog.services.requests.get')
    def test_api_error_handling(self, mock_get):
        """Test handling of API errors."""
        mock_get.side_effect = requests.RequestException('Connection failed')
        
        from blog.services import fetch_external_articles
        result = fetch_external_articles()
        
        self.assertEqual(result, [])  # Returns empty on error
```

## Mocking Datetime

```python
from freezegun import freeze_time  # pip install freezegun


class TimeBasedTests(TestCase):
    @freeze_time('2024-01-15 12:00:00')
    def test_article_is_new(self):
        """Test article is considered 'new' within 7 days."""
        article = Article.objects.create(
            title='Test',
            pub_date=timezone.datetime(2024, 1, 10, tzinfo=timezone.utc)
        )
        
        self.assertTrue(article.is_new)  # 5 days old
    
    @freeze_time('2024-01-15 12:00:00')
    def test_article_not_new(self):
        """Test article is not 'new' after 7 days."""
        article = Article.objects.create(
            title='Test',
            pub_date=timezone.datetime(2024, 1, 1, tzinfo=timezone.utc)
        )
        
        self.assertFalse(article.is_new)  # 14 days old
```

## Context Manager Patching

```python
class ContextPatchTests(TestCase):
    def test_with_context_manager(self):
        """Use patch as context manager."""
        with patch('blog.views.cache.get') as mock_cache_get:
            mock_cache_get.return_value = None  # Cache miss
            
            response = self.client.get('/articles/')
            
            # Verify cache was checked
            mock_cache_get.assert_called()
    
    def test_multiple_patches(self):
        """Patch multiple things."""
        with patch('blog.views.cache.get') as mock_get, \
             patch('blog.views.cache.set') as mock_set:
            
            mock_get.return_value = None
            
            response = self.client.get('/articles/')
            
            # Cache miss triggers cache set
            mock_set.assert_called_once()
```

## Mock Objects

```python
class MockObjectTests(TestCase):
    def test_mock_model(self):
        """Create mock model objects."""
        mock_user = Mock(spec=User)
        mock_user.username = 'testuser'
        mock_user.email = 'test@example.com'
        mock_user.is_authenticated = True
        
        # Use in test
        from blog.utils import format_author_name
        result = format_author_name(mock_user)
        self.assertEqual(result, 'testuser')
    
    def test_magic_mock(self):
        """MagicMock supports magic methods."""
        mock_qs = MagicMock()
        mock_qs.__len__.return_value = 5
        mock_qs.__iter__.return_value = iter([1, 2, 3, 4, 5])
        
        self.assertEqual(len(mock_qs), 5)
        self.assertEqual(list(mock_qs), [1, 2, 3, 4, 5])
```

## Resources

- [unittest.mock](https://docs.python.org/3/library/unittest.mock.html) — Python mocking library

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*