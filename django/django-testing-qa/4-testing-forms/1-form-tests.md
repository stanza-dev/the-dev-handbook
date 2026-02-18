---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-form-tests"
---

# Form Testing

Test form validation, cleaning, and rendering.

## Testing Form Validity

```python
from django.test import TestCase
from .forms import ArticleForm, ContactForm


class ArticleFormTests(TestCase):
    def test_valid_form(self):
        """Test form with valid data."""
        form = ArticleForm(data={
            'title': 'Test Article',
            'body': 'This is the body of the article.',
            'status': 'draft',
        })
        
        self.assertTrue(form.is_valid())
    
    def test_invalid_form_missing_title(self):
        """Test form with missing required field."""
        form = ArticleForm(data={
            'body': 'Content',
            'status': 'draft',
        })
        
        self.assertFalse(form.is_valid())
        self.assertIn('title', form.errors)
        self.assertEqual(
            form.errors['title'][0],
            'This field is required.'
        )
    
    def test_form_field_count(self):
        """Test that form has expected fields."""
        form = ArticleForm()
        expected_fields = ['title', 'body', 'status', 'categories']
        
        self.assertEqual(list(form.fields.keys()), expected_fields)
```

## Testing Custom Validation

```python
class ArticleFormValidationTests(TestCase):
    def test_title_too_short(self):
        """Test custom title length validation."""
        form = ArticleForm(data={
            'title': 'Hi',  # Too short
            'body': 'Content',
            'status': 'draft',
        })
        
        self.assertFalse(form.is_valid())
        self.assertIn('title', form.errors)
        self.assertIn('at least 5 characters', form.errors['title'][0])
    
    def test_clean_method_validation(self):
        """Test form-level validation in clean()."""
        form = ArticleForm(data={
            'title': 'Test Article',
            'body': 'Content',
            'status': 'published',
            'pub_date': None,  # Required for published
        })
        
        self.assertFalse(form.is_valid())
        self.assertIn('__all__', form.errors)  # Non-field error
    
    def test_profanity_filter(self):
        """Test that profanity is rejected."""
        form = ArticleForm(data={
            'title': 'Bad Word Article',
            'body': 'Contains badword here',
            'status': 'draft',
        })
        
        self.assertFalse(form.is_valid())
        self.assertIn('body', form.errors)
```

## Testing Form Save

```python
class ArticleFormSaveTests(TestCase):
    def setUp(self):
        self.user = User.objects.create_user('test', 'test@test.com', 'pass')
    
    def test_form_save(self):
        """Test that form saves correctly."""
        form = ArticleForm(data={
            'title': 'Test Article',
            'body': 'Content',
            'status': 'draft',
        })
        
        self.assertTrue(form.is_valid())
        article = form.save(commit=False)
        article.author = self.user
        article.save()
        
        self.assertEqual(Article.objects.count(), 1)
        self.assertEqual(article.author, self.user)
    
    def test_form_update(self):
        """Test form updates existing object."""
        article = Article.objects.create(
            title='Original',
            body='Original content',
            author=self.user
        )
        
        form = ArticleForm(
            data={'title': 'Updated', 'body': 'New content', 'status': 'draft'},
            instance=article
        )
        
        self.assertTrue(form.is_valid())
        form.save()
        
        article.refresh_from_db()
        self.assertEqual(article.title, 'Updated')
```

## Testing Formsets

```python
from django.forms import formset_factory
from .forms import ImageForm


class ImageFormsetTests(TestCase):
    def test_valid_formset(self):
        """Test formset with valid data."""
        ImageFormset = formset_factory(ImageForm, extra=2)
        
        data = {
            'form-TOTAL_FORMS': '2',
            'form-INITIAL_FORMS': '0',
            'form-0-url': 'http://example.com/image1.jpg',
            'form-0-caption': 'Image 1',
            'form-1-url': 'http://example.com/image2.jpg',
            'form-1-caption': 'Image 2',
        }
        
        formset = ImageFormset(data=data)
        self.assertTrue(formset.is_valid())
        self.assertEqual(len(formset.forms), 2)
    
    def test_formset_min_num(self):
        """Test formset minimum forms validation."""
        ImageFormset = formset_factory(ImageForm, min_num=1, validate_min=True)
        
        data = {
            'form-TOTAL_FORMS': '1',
            'form-INITIAL_FORMS': '0',
            'form-0-url': '',
            'form-0-caption': '',
        }
        
        formset = ImageFormset(data=data)
        self.assertFalse(formset.is_valid())
```

## Testing Form Rendering

```python
class FormRenderingTests(TestCase):
    def test_form_html_output(self):
        """Test form renders expected HTML."""
        form = ArticleForm()
        html = form.as_p()
        
        self.assertIn('name="title"', html)
        self.assertIn('name="body"', html)
        self.assertIn('<label', html)
    
    def test_form_widget_attributes(self):
        """Test custom widget attributes."""
        form = ArticleForm()
        
        title_field = form.fields['title']
        self.assertEqual(
            title_field.widget.attrs.get('class'),
            'form-control'
        )
        self.assertEqual(
            title_field.widget.attrs.get('placeholder'),
            'Enter article title'
        )
```

## Resources

- [Form Testing](https://docs.djangoproject.com/en/6.0/topics/testing/tools/#django.test.SimpleTestCase.assertFormError) — Testing forms in Django

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*