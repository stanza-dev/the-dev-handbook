---
source_course: "django-rest-api"
source_lesson: "django-rest-api-idempotency"
---

# Idempotency in APIs

## Introduction

Network failures happen. When a client doesn't receive a response, should they retry? Idempotent APIs handle retries gracefully without causing duplicate effects.

## Key Concepts

**Idempotency**: Making the same request multiple times has the same effect as making it once.

**Idempotency Key**: Client-generated unique ID to identify duplicate requests.

**Safe Methods**: GET, HEAD, OPTIONS don't modify data—inherently safe to retry.

## Deep Dive

### Idempotent HTTP Methods

```
GET    - Idempotent (no side effects)
PUT    - Idempotent (same result each time)
DELETE - Idempotent (resource stays deleted)
POST   - NOT idempotent (creates duplicates)
PATCH  - Usually idempotent
```

### Idempotency Keys for POST

```python
def create_order(request):
    idempotency_key = request.headers.get('Idempotency-Key')
    
    if not idempotency_key:
        return api_error('MISSING_IDEMPOTENCY_KEY', 'Idempotency-Key header required', status=400)
    
    # Check for existing request with this key
    cache_key = f'idempotency:{request.user.id}:{idempotency_key}'
    cached_response = cache.get(cache_key)
    
    if cached_response:
        return JsonResponse(cached_response)  # Return same response
    
    # Process the request
    order = Order.objects.create(
        user=request.user,
        idempotency_key=idempotency_key,
        **order_data
    )
    
    response_data = {'id': order.id, 'status': 'created'}
    cache.set(cache_key, response_data, 86400)  # Cache for 24 hours
    
    return JsonResponse(response_data, status=201)
```

### Database-Level Uniqueness

```python
# models.py
class Order(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    idempotency_key = models.CharField(max_length=64)
    # ...
    
    class Meta:
        unique_together = ['user', 'idempotency_key']

# views.py
from django.db import IntegrityError

def create_order(request):
    try:
        order = Order.objects.create(
            user=request.user,
            idempotency_key=data['idempotency_key'],
            **order_data
        )
    except IntegrityError:
        # Duplicate - return existing order
        order = Order.objects.get(
            user=request.user,
            idempotency_key=data['idempotency_key']
        )
    
    return JsonResponse({'id': order.id})
```

## Best Practices

1. **Require idempotency keys for POST**: Especially for payments and important operations.
2. **Store results**: Cache or database to return consistent responses.
3. **Time-limited keys**: Expire old keys to prevent storage bloat.
4. **Document clearly**: Explain idempotency behavior in API docs.

## Summary

Idempotency ensures safe retries. GET/PUT/DELETE are naturally idempotent; for POST, implement idempotency keys that let clients safely retry requests without creating duplicates.

## Resources

- [Idempotency Patterns](https://stripe.com/docs/api/idempotent_requests) — Stripe's idempotency implementation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*