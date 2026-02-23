---
source_course: "django-rest-api"
source_lesson: "django-rest-api-validation"
---

# Input Validation

## Introduction

Never trust user input. This fundamental security principle applies especially to APIs, where data comes from potentially untrusted sources. Proper validation ensures data integrity, prevents security vulnerabilities, and provides helpful feedback to API consumers.

## Key Concepts

**Validation**: The process of checking that input data meets expected criteria before processing it.

**Sanitization**: Cleaning input data by removing or escaping potentially harmful content.

**Django Forms**: Django's built-in validation framework that works equally well for API data as for HTML forms.

**Cleaned Data**: Validated and normalized data ready for use, accessed via `form.cleaned_data`.

## Real World Context

In production APIs, validation prevents:
- **SQL injection**: Malicious data in database queries
- **XSS attacks**: Script injection through user content
- **Data corruption**: Invalid data types breaking your application
- **Business logic violations**: Orders with negative quantities, invalid dates

Without validation, a single malformed request can crash your server or corrupt your database.

## Deep Dive

### Using Django Forms for Validation

Django's `ModelForm` provides built-in validation that works well for API input. Define a form with your model and add custom `clean_` methods for field-specific rules:

```python
# forms.py
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'body', 'category']
    
    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError('Title must be at least 5 characters')
        if Article.objects.filter(title=title).exists():
            raise forms.ValidationError('An article with this title already exists')
        return title
```

```python
# views.py
def api_create_article(request):
    data = json.loads(request.body)
    form = ArticleForm(data)
    
    if form.is_valid():
        article = form.save(commit=False)
        article.author = request.user
        article.save()
        return JsonResponse({'id': article.id}, status=201)
    
    return JsonResponse({'errors': form.errors}, status=400)
```

When `form.is_valid()` returns `False`, `form.errors` is a dictionary mapping field names to lists of error messages -- ready to return directly as JSON.

### Custom Validator Class

For cases where Django forms are too heavyweight or your validation rules don't map to a model, you can build a lightweight validator class:

```python
class ArticleValidator:
    def __init__(self, data):
        self.data = data
        self.errors = {}
    
    def is_valid(self):
        self.errors = {}
        self._validate_title()
        self._validate_body()
        return len(self.errors) == 0
    
    def _validate_title(self):
        title = self.data.get('title')
        if not title:
            self.errors['title'] = ['This field is required']
        elif len(title) < 5:
            self.errors['title'] = ['Must be at least 5 characters']
    
    def _validate_body(self):
        body = self.data.get('body', '')
        if len(body) > 50000:
            self.errors['body'] = ['Content too long (max 50000 chars)']
    
    @property
    def cleaned_data(self):
        return {
            'title': self.data.get('title', '').strip(),
            'body': self.data.get('body', '').strip(),
        }
```

The `cleaned_data` property returns sanitized values (e.g., stripped whitespace), separating validation from data cleaning.

### Validation Decorator

A decorator can handle JSON parsing and required-field checks in one reusable wrapper, keeping your view functions focused on business logic:

```python
from functools import wraps

def validate_json(required_fields=None):
    required_fields = required_fields or []
    
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            try:
                data = json.loads(request.body)
            except json.JSONDecodeError:
                return JsonResponse({'error': 'Invalid JSON'}, status=400)
            
            missing = [f for f in required_fields if f not in data]
            if missing:
                return JsonResponse({'error': 'Missing fields', 'fields': missing}, status=400)
            
            request.json_data = data
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator

@validate_json(required_fields=['title'])
def api_create_article(request):
    data = request.json_data
    article = Article.objects.create(title=data['title'])
    return JsonResponse({'id': article.id}, status=201)
```

The decorator attaches the parsed data to `request.json_data`, so the view can access it without repeating the parsing and validation logic.

## Common Pitfalls

1. **Validating only on the frontend**: Client-side validation improves UX but can be bypassed. Always validate on the server.

2. **Generic error messages**: `{'error': 'Invalid data'}` doesn't help developers fix their requests. Be specific about which fields failed and why.

3. **Forgetting edge cases**: Empty strings, null values, extremely long inputs, special characters—test them all.

## Best Practices

1. **Validate early, fail fast**: Check input before any database operations.

2. **Use Django forms**: They handle many edge cases automatically and integrate with model validation.

3. **Return field-level errors**: Structure errors as `{'field_name': ['error1', 'error2']}`.

4. **Sanitize after validation**: Strip whitespace, normalize Unicode, escape HTML as needed.

5. **Set reasonable limits**: Max lengths, allowed characters, value ranges.

## Summary

Input validation is your first line of defense. Use Django forms for comprehensive validation, create custom validators for complex business rules, and always return helpful, field-level error messages. Remember: validate on the server, be specific about errors, and never trust client data.

## Code Examples

**Input validation using Django ModelForm in an API view**

```python
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'body', 'category']

    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError('Title must be at least 5 characters')
        return title

def api_create_article(request):
    data = json.loads(request.body)
    form = ArticleForm(data)
    if form.is_valid():
        article = form.save()
        return JsonResponse({'id': article.id}, status=201)
    return JsonResponse({'errors': form.errors}, status=400)
```


## Resources

- [Form and Field Validation](https://docs.djangoproject.com/en/6.0/ref/forms/validation/) — Official guide to form validation
- [Validators Reference](https://docs.djangoproject.com/en/6.0/ref/validators/) — Built-in Django validators

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*