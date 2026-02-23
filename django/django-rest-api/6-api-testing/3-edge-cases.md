---
source_course: "django-rest-api"
source_lesson: "django-rest-api-test-edge-cases"
---

# Testing Edge Cases

## Introduction

Edge cases are where bugs hide. Testing boundary conditions, error scenarios, and unusual inputs catches issues before they reach production.

## Key Concepts

**Edge Case**: Unusual or boundary conditions that may cause unexpected behavior.

**Boundary Testing**: Testing at the limits of valid input (empty, max length, zero, negative).

**Error Path Testing**: Verifying proper handling when things go wrong.

## Real World Context

Edge case testing catches real-world bugs:
- **Unicode input**: Users submit names, comments, and content in any language
- **Concurrent requests**: Multiple users updating the same resource simultaneously
- **Boundary values**: Empty strings, maximum lengths, zero quantities, negative numbers
- **Invalid content types**: Clients sending XML when JSON is expected

## Deep Dive

### Testing Validation Errors

Test every validation rule by submitting data that violates it and verifying the API returns the correct error response:

```python
class ArticleValidationTests(TestCase):
    def test_create_without_title(self):
        response = self.client.post('/api/articles/',
            data={'body': 'Content'},
            content_type='application/json'
        )
        self.assertEqual(response.status_code, 400)
        self.assertIn('title', response.json()['errors'])
    
    def test_title_too_long(self):
        response = self.client.post('/api/articles/',
            data={'title': 'x' * 300},
            content_type='application/json'
        )
        self.assertEqual(response.status_code, 400)
    
    def test_empty_string_title(self):
        response = self.client.post('/api/articles/',
            data={'title': '', 'body': 'Content'},
            content_type='application/json'
        )
        self.assertEqual(response.status_code, 400)
```

Each test targets one specific validation case: missing required field, exceeding max length, or empty string input. This makes test failures easy to diagnose.

### Testing Authentication/Authorization

Authorization tests verify that unauthenticated requests, unauthorized users, and invalid tokens are all rejected with the correct status codes:

```python
class AuthorizationTests(TestCase):
    def test_unauthenticated_access(self):
        response = self.client.get('/api/profile/')
        self.assertEqual(response.status_code, 401)
    
    def test_wrong_user_cannot_edit(self):
        owner = UserFactory()
        other = UserFactory()
        article = ArticleFactory(author=owner)
        
        self.authenticate_as(other)
        response = self.client.put(f'/api/articles/{article.id}/',
            data={'title': 'Hacked'},
            content_type='application/json'
        )
        self.assertEqual(response.status_code, 403)
    
    def test_invalid_token(self):
        response = self.client.get('/api/profile/',
            HTTP_AUTHORIZATION='Bearer invalid-token'
        )
        self.assertEqual(response.status_code, 401)
```

The `test_wrong_user_cannot_edit` test creates two separate users and verifies that one cannot modify the other's article -- a common authorization bug in CRUD APIs.

### Testing Pagination Boundaries

Pagination boundary tests verify correct behavior for invalid page values like zero, negative numbers, out-of-range pages, and non-numeric input:

```python
class PaginationEdgeCases(TestCase):
    def test_page_zero(self):
        response = self.client.get('/api/articles/?page=0')
        self.assertEqual(response.status_code, 200)  # Should default to page 1
    
    def test_negative_page(self):
        response = self.client.get('/api/articles/?page=-1')
        self.assertIn(response.status_code, [200, 400])
    
    def test_page_beyond_total(self):
        ArticleFactory.create_batch(5)
        response = self.client.get('/api/articles/?page=999')
        self.assertEqual(response.json()['data'], [])
    
    def test_invalid_page_type(self):
        response = self.client.get('/api/articles/?page=abc')
        self.assertEqual(response.status_code, 400)
```

These tests document the expected behavior for ambiguous cases. Whether page 0 returns page 1 or an error is a design decision -- the test locks it in.

## Common Pitfalls

1. **Only testing valid input**: Real clients send empty strings, null values, extremely long inputs, and special characters.

2. **Ignoring authentication edge cases**: Expired tokens, revoked permissions, deleted users.

3. **Not testing pagination boundaries**: Page 0, negative pages, pages beyond the total, non-numeric page values.

## Best Practices

1. **Test the unhappy path**: Errors, invalid input, edge cases.
2. **Test boundaries**: Empty, zero, negative, max values.
3. **Test concurrent scenarios**: Race conditions, duplicate submissions.

## Summary

Edge case testing catches bugs that happy-path tests miss. Test validation errors, authentication failures, pagination boundaries, and unusual inputs to build robust APIs.

## Code Examples

**Testing validation edge cases with boundary inputs**

```python
class ArticleValidationTests(TestCase):
    def test_create_without_title(self):
        response = self.client.post('/api/articles/',
            data={'body': 'Content'},
            content_type='application/json')
        self.assertEqual(response.status_code, 400)
        self.assertIn('title', response.json()['errors'])

    def test_title_too_long(self):
        response = self.client.post('/api/articles/',
            data={'title': 'x' * 300},
            content_type='application/json')
        self.assertEqual(response.status_code, 400)

    def test_empty_string_title(self):
        response = self.client.post('/api/articles/',
            data={'title': '', 'body': 'Content'},
            content_type='application/json')
        self.assertEqual(response.status_code, 400)
```


## Resources

- [Django Testing Overview](https://docs.djangoproject.com/en/6.0/topics/testing/overview/) — Django testing concepts

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*