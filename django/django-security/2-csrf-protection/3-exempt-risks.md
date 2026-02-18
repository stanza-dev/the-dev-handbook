---
source_course: "django-security"
source_lesson: "django-security-csrf-exempt-risks"
---

# CSRF Exemption Risks

## Introduction

The @csrf_exempt decorator is tempting for APIs and webhooks, but improper use creates serious vulnerabilities. Learn when exemption is safe and when it's not.

## Key Concepts

**csrf_exempt**: Decorator that disables CSRF protection for a view.

**Webhook Verification**: Alternative security for external service callbacks.

**Token Authentication**: Auth method that doesn't require CSRF protection.

## Deep Dive

### When CSRF Exemption is SAFE

```python
# 1. Token-authenticated APIs (no cookies used)
@api_view(['POST'])
@authentication_classes([TokenAuthentication])
def api_endpoint(request):
    pass  # Token auth doesn't use cookies

# 2. Webhooks with signature verification
@csrf_exempt
def stripe_webhook(request):
    payload = request.body
    sig = request.headers.get('Stripe-Signature')
    
    try:
        event = stripe.Webhook.construct_event(
            payload, sig, webhook_secret
        )
    except stripe.error.SignatureVerificationError:
        return HttpResponse(status=400)
    
    # Process verified webhook
    return HttpResponse(status=200)
```

### When CSRF Exemption is DANGEROUS

```python
# DANGEROUS: Session auth + csrf_exempt
@csrf_exempt
def transfer_money(request):
    if request.user.is_authenticated:  # Uses session cookie
        # Attacker can forge this request!
        transfer(request.user, request.POST['amount'], request.POST['to'])
```

### Safe Pattern: Verify Webhooks

```python
import hmac
import hashlib

def verify_webhook(request, secret):
    signature = request.headers.get('X-Webhook-Signature')
    expected = hmac.new(
        secret.encode(),
        request.body,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(signature, expected)
```

## Best Practices

1. **Never exempt session-authenticated views**: Use CSRF or switch to token auth.
2. **Always verify webhooks**: Use signatures, not just @csrf_exempt.
3. **Document exemptions**: Comment why each exemption is safe.

## Summary

@csrf_exempt is safe ONLY when not using session authentication. For webhooks, always verify signatures. For APIs, use token authentication which doesn't need CSRF protection.

## Resources

- [CSRF Decorator](https://docs.djangoproject.com/en/6.0/ref/csrf/#django.views.decorators.csrf.csrf_exempt) — csrf_exempt decorator reference

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*