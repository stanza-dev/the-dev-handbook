---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-test-organization"
---

# Test Organization and Structure

## Introduction

Well-organized tests are easier to maintain and understand. A good structure helps teams collaborate and ensures comprehensive coverage.

## Key Concepts

**Test Module**: A Python file containing test classes.

**Test Class**: Groups related tests together.

**Test Method**: Individual test case starting with `test_`.

## Deep Dive

### Directory Structure

```
myapp/
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_views.py
│   ├── test_forms.py
│   ├── test_api.py
│   └── factories.py
├── models.py
├── views.py
└── forms.py
```

### Naming Conventions

```python
class ArticleModelTests(TestCase):
    """Tests for the Article model."""
    
    def test_article_creation_with_valid_data(self):
        """Test that an article can be created with valid data."""
        pass
    
    def test_article_str_returns_title(self):
        """Test the __str__ method returns the title."""
        pass
    
    def test_article_slug_is_unique(self):
        """Test that duplicate slugs raise an error."""
        pass
```

### Using Docstrings

```python
class ArticleViewTests(TestCase):
    """
    Tests for article views.
    
    These tests cover:
    - List view pagination
    - Detail view 404 handling
    - Create view permissions
    """
    
    def test_list_view_returns_200(self):
        """GET /articles/ returns 200 OK."""
        pass
```

## Real World Context

In large Django projects with hundreds of tests, good organization is the difference between a maintainable test suite and one that nobody wants to touch. Teams that mirror their app structure in tests can quickly find and update tests when code changes. The AAA (Arrange, Act, Assert) pattern makes tests readable even for new team members.

## Common Pitfalls

1. **Putting all tests in one file**: A single `tests.py` becomes unmanageable fast. Split into `test_models.py`, `test_views.py`, etc.
2. **Vague test names**: `test_article` tells you nothing. Use `test_article_creation_with_valid_data` to describe the expected behavior.
3. **Multiple concerns per test**: A test that checks creation, update, and deletion is three tests in disguise.

## Best Practices

1. **One assertion per test**: Each test should verify one behavior.
2. **Descriptive names**: Test names should describe expected behavior.
3. **AAA pattern**: Arrange, Act, Assert structure.
4. **Keep tests fast**: Avoid slow operations in tests.

## Summary

Organize tests by feature or model. Use descriptive names and docstrings. Follow the AAA pattern and keep tests focused on single behaviors.

## Resources

- [Testing Best Practices](https://docs.djangoproject.com/en/6.0/topics/testing/overview/) — Django testing overview

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*