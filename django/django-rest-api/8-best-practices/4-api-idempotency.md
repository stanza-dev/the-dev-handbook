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

## Real World Context

Idempotency is critical for:
- **Payment processing**: Stripe requires idempotency keys to prevent duplicate charges
- **Order placement**: E-commerce APIs must prevent duplicate orders from retries
- **Webhook delivery**: Webhook receivers must handle duplicate deliveries gracefully
- **Distributed systems**: Network partitions and retries are common in microservice architectures

## Deep Dive

### Idempotent HTTP Methods

```
GET    - Idempotent (no side effects)
PUT    - Idempotent (same result each time)
DELETE - Idempotent (resource stays deleted)
POST   - NOT idempotent (creates duplicates)
PATCH  - NOT guaranteed idempotent (depends on patch format)
```

### Idempotency Keys for POST

Clients include an `Idempotency-Key` header with a unique value (typically a UUID). The server caches the response and returns it unchanged on retry:

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

The cache TTL of 24 hours means clients can safely retry within that window. After 24 hours, the key expires and can be reused.

### Database-Level Uniqueness

For critical operations like payments, back the idempotency key with a database unique constraint for durability beyond cache TTL:

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

The `unique_together` constraint on `(user, idempotency_key)` catches duplicates at the database level, even if the cache entry has expired. The `IntegrityError` handler returns the existing order instead of creating a duplicate.

## Common Pitfalls

1. **Not requiring idempotency keys for mutations**: POST endpoints that create resources should require idempotency keys.

2. **Infinite key storage**: Idempotency keys should expire (24-48 hours). Don't store them forever.

3. **Caching only the response, not the status code**: A cached 201 response should be returned with the same 201 status, not 200.

## Best Practices

1. **Require idempotency keys for POST**: Especially for payments and important operations.
2. **Store results**: Cache or database to return consistent responses.
3. **Time-limited keys**: Expire old keys to prevent storage bloat.
4. **Document clearly**: Explain idempotency behavior in API docs.

## Summary

Idempotency ensures safe retries. GET/PUT/DELETE are naturally idempotent; for POST, implement idempotency keys that let clients safely retry requests without creating duplicates.

## Code Examples

**Idempotency key implementation to prevent duplicate operations**

```python
from django.core.cache import cache

def create_order(request):
    idempotency_key = request.headers.get('Idempotency-Key')
    if not idempotency_key:
        return JsonResponse({'error': 'Idempotency-Key required'}, status=400)

    cache_key = f'idempotency:{request.user.id}:{idempotency_key}'
    cached = cache.get(cache_key)
    if cached:
        return JsonResponse(cached)  # Return same response

    order = Order.objects.create(user=request.user, **order_data)
    response_data = {'id': order.id, 'status': 'created'}
    cache.set(cache_key, response_data, 86400)
    return JsonResponse(response_data, status=201)
```


## Resources

- [Idempotency Patterns](https://stripe.com/docs/api/idempotent_requests) — Stripe's idempotency implementation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*