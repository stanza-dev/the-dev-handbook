---
source_course: "django-rest-api"
source_lesson: "django-rest-api-handling-request-data"
---

# Handling Request Data

## Introduction

APIs are a two-way street. While returning JSON responses is essential, equally important is receiving and processing data from clients. Whether it's a user submitting a form, a mobile app uploading a photo, or a service sending webhook data, your API needs to handle incoming data safely and efficiently.

## Key Concepts

**Request Body**: The payload sent with POST, PUT, and PATCH requests. In REST APIs, this is typically JSON data accessed via `request.body`.

**Query Parameters**: Key-value pairs in the URL (e.g., `?page=2&search=django`), accessed via `request.GET`.

**Request Headers**: Metadata sent with requests like `Authorization`, `Content-Type`, and custom headers, accessed via `request.headers`.

**CSRF (Cross-Site Request Forgery)**: Django's built-in protection against malicious requests. API endpoints often use token authentication instead.

## Real World Context

Every production API handles:
- **User registration**: Parsing email, password, profile data from POST requests
- **Search and filtering**: Reading query parameters for pagination, sorting, filtering
- **File uploads**: Processing multipart form data
- **Webhooks**: Receiving and validating data from external services (Stripe, GitHub, etc.)

Poor request handling leads to security vulnerabilities, data corruption, and frustrating error messages for API consumers.

## Deep Dive

### Parsing JSON Request Body

```python
import json
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt  # Use token auth instead for APIs
def api_create_article(request):
    if request.method != 'POST':
        return JsonResponse({'error': 'Method not allowed'}, status=405)
    
    # Parse JSON body
    try:
        data = json.loads(request.body)
    except json.JSONDecodeError:
        return JsonResponse({'error': 'Invalid JSON'}, status=400)
    
    # Validate required fields
    if 'title' not in data:
        return JsonResponse(
            {'error': 'Validation failed', 'details': {'title': 'This field is required'}},
            status=400
        )
    
    # Create the resource
    article = Article.objects.create(
        title=data['title'],
        body=data.get('body', ''),
        author=request.user
    )
    
    return JsonResponse({'id': article.id, 'title': article.title}, status=201)
```

### Query Parameters

```python
def api_articles(request):
    # GET /api/articles?page=2&per_page=10&search=django&sort=-created_at
    
    # Parse with defaults and type conversion
    try:
        page = int(request.GET.get('page', 1))
        per_page = min(int(request.GET.get('per_page', 20)), 100)  # Cap at 100
    except ValueError:
        return JsonResponse({'error': 'Invalid pagination parameters'}, status=400)
    
    search = request.GET.get('search', '').strip()
    sort = request.GET.get('sort', '-created_at')
    
    # Build queryset
    articles = Article.objects.all()
    
    if search:
        articles = articles.filter(title__icontains=search)
    
    # Validate sort field
    allowed_sorts = ['created_at', '-created_at', 'title', '-title']
    if sort in allowed_sorts:
        articles = articles.order_by(sort)
    
    # Paginate
    start = (page - 1) * per_page
    total = articles.count()
    articles = articles[start:start + per_page]
    
    return JsonResponse({
        'data': list(articles.values('id', 'title', 'created_at')),
        'meta': {'page': page, 'per_page': per_page, 'total': total}
    })
```

### Complete CRUD View

```python
@csrf_exempt
def api_article(request, pk=None):
    if request.method == 'GET':
        if pk:
            article = get_object_or_404(Article, pk=pk)
            return JsonResponse({'id': article.id, 'title': article.title, 'body': article.body})
        articles = Article.objects.all()[:20]
        return JsonResponse({'data': list(articles.values('id', 'title'))})
    
    elif request.method == 'POST':
        data = json.loads(request.body)
        article = Article.objects.create(title=data['title'], body=data.get('body', ''))
        return JsonResponse({'id': article.id}, status=201)
    
    elif request.method == 'PUT':  # Full replacement
        article = get_object_or_404(Article, pk=pk)
        data = json.loads(request.body)
        article.title = data['title']  # Required
        article.body = data['body']    # Required
        article.save()
        return JsonResponse({'id': article.id})
    
    elif request.method == 'PATCH':  # Partial update
        article = get_object_or_404(Article, pk=pk)
        data = json.loads(request.body)
        if 'title' in data:
            article.title = data['title']
        if 'body' in data:
            article.body = data['body']
        article.save()
        return JsonResponse({'id': article.id})
    
    elif request.method == 'DELETE':
        article = get_object_or_404(Article, pk=pk)
        article.delete()
        return JsonResponse({}, status=204)
    
    return JsonResponse({'error': 'Method not allowed'}, status=405)
```

### Reading Request Headers

```python
def api_protected(request):
    # Access headers (case-insensitive)
    auth = request.headers.get('Authorization', '')
    content_type = request.headers.get('Content-Type')
    api_key = request.headers.get('X-API-Key')
    
    if not api_key or api_key != settings.API_KEY:
        return JsonResponse({'error': 'Invalid API key'}, status=401)
```

## Common Pitfalls

1. **Not handling JSONDecodeError**: Always wrap `json.loads()` in try/except. Malformed JSON should return 400, not 500.

2. **Type coercion failures**: `int(request.GET.get('page'))` crashes if page is missing or not a number. Always provide defaults and handle ValueError.

3. **Trusting client data**: Never use `data.get('is_admin', False)` to set permissions. Validate and sanitize all input against your business rules.

## Best Practices

1. **Validate early, fail fast**: Check required fields before any database operations.

2. **Return helpful error messages**: Include field-level details: `{'errors': {'email': 'Invalid format'}}`.

3. **Cap pagination limits**: Prevent `?per_page=1000000` from killing your server.

4. **Whitelist allowed query parameters**: Don't let users sort or filter by arbitrary fields.

5. **Use consistent parameter names**: Stick to `page`/`per_page` or `offset`/`limit` across all endpoints.

## Summary

Handling request data involves parsing JSON bodies with `json.loads(request.body)`, reading query parameters from `request.GET`, and accessing headers via `request.headers`. Always validate input, handle parsing errors gracefully, and return meaningful error messages. Remember that PUT expects complete resource replacement while PATCH handles partial updates.

## Resources

- [HttpRequest Objects](https://docs.djangoproject.com/en/6.0/ref/request-response/#httprequest-objects) — Complete HttpRequest reference
- [View Decorators](https://docs.djangoproject.com/en/6.0/topics/http/decorators/) — Django view decorators including csrf_exempt

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*