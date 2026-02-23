---
source_course: "django-deployment"
source_lesson: "django-deployment-logging-configuration"
---

# Production Logging

## Introduction

Proper logging is the backbone of production observability. Django's logging framework, built on Python's `logging` module, supports multiple handlers, formatters, and log levels. Combined with error tracking tools like Sentry, it enables rapid debugging of production issues.

## Key Concepts

- **Logger**: A named channel that application code uses to emit log messages (e.g., `logging.getLogger(__name__)`).
- **Handler**: A destination for log messages (console, file, email, external service).
- **Formatter**: Defines the format of log output (text, JSON for structured logging).
- **Sentry**: An error tracking platform that captures exceptions with full stack traces and context.

## Real World Context

Production Django applications typically use JSON-formatted logging for machine parseability, sent to stdout for container-based deployments. Log aggregation services (ELK stack, Datadog, CloudWatch) collect and index these logs for searching and alerting. Sentry integration captures unhandled exceptions automatically, providing stack traces, request data, and user context that make debugging 10x faster than reading raw logs.

## Deep Dive

Production logging must be structured, rotated, and centralized. Django's logging configuration uses Python's standard logging module with dictConfig format.


## Introduction

Proper logging is essential for debugging and monitoring production systems.

## Django Logging Configuration

```python
# settings/production.py

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'json': {
            '()': 'pythonjsonlogger.jsonlogger.JsonFormatter',
            'format': '%(asctime)s %(levelname)s %(name)s %(message)s',
        },
    },
    
    'filters': {
        'require_debug_false': {
            '()': 'django.utils.log.RequireDebugFalse',
        },
    },
    
    'handlers': {
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'json',
        },
        'file': {
            'level': 'WARNING',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '/var/log/django/myproject.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 5,
            'formatter': 'verbose',
        },
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
            'filters': ['require_debug_false'],
        },
    },
    
    'root': {
        'handlers': ['console'],
        'level': 'INFO',
    },
    
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': 'INFO',
            'propagate': False,
        },
        'django.request': {
            'handlers': ['console', 'file', 'mail_admins'],
            'level': 'WARNING',
            'propagate': False,
        },
        'myapp': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
```

## Using Loggers in Code

```python
import logging

logger = logging.getLogger(__name__)


def process_order(order_id):
    logger.info(f'Processing order {order_id}')
    
    try:
        order = Order.objects.get(pk=order_id)
        logger.debug(f'Order details: {order}')
        
        result = charge_payment(order)
        logger.info(f'Payment successful for order {order_id}')
        
        return result
        
    except Order.DoesNotExist:
        logger.error(f'Order {order_id} not found')
        raise
        
    except PaymentError as e:
        logger.exception(f'Payment failed for order {order_id}')
        raise
```

## Sentry Error Tracking

```python
# pip install sentry-sdk

# settings/production.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration
from sentry_sdk.integrations.celery import CeleryIntegration
from sentry_sdk.integrations.redis import RedisIntegration

sentry_sdk.init(
    dsn=os.environ['SENTRY_DSN'],
    integrations=[
        DjangoIntegration(),
        CeleryIntegration(),
        RedisIntegration(),
    ],
    traces_sample_rate=0.1,  # 10% of transactions
    profiles_sample_rate=0.1,
    send_default_pii=False,
    environment=os.environ.get('ENVIRONMENT', 'production'),
)
```

## Health Check Endpoint

```python
# views.py
from django.http import JsonResponse
from django.db import connection
from django.core.cache import cache


def health_check(request):
    """Health check endpoint for load balancers/monitoring."""
    health = {
        'status': 'healthy',
        'checks': {}
    }
    
    # Database check
    try:
        with connection.cursor() as cursor:
            cursor.execute('SELECT 1')
        health['checks']['database'] = 'ok'
    except Exception as e:
        health['checks']['database'] = str(e)
        health['status'] = 'unhealthy'
    
    # Cache check
    try:
        cache.set('health_check', 'ok', 10)
        if cache.get('health_check') == 'ok':
            health['checks']['cache'] = 'ok'
        else:
            health['checks']['cache'] = 'failed'
            health['status'] = 'unhealthy'
    except Exception as e:
        health['checks']['cache'] = str(e)
        health['status'] = 'unhealthy'
    
    status_code = 200 if health['status'] == 'healthy' else 503
    return JsonResponse(health, status=status_code)
```

## Prometheus Metrics

```python
# pip install django-prometheus

# settings.py
INSTALLED_APPS = [
    # ...
    'django_prometheus',
]

MIDDLEWARE = [
    'django_prometheus.middleware.PrometheusBeforeMiddleware',
    # ... other middleware ...
    'django_prometheus.middleware.PrometheusAfterMiddleware',
]

# urls.py
urlpatterns = [
    path('', include('django_prometheus.urls')),
]
```

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Logging sensitive data** — Never log passwords, API keys, credit card numbers, or personally identifiable information. Use Sentry's `send_default_pii=False` setting.
2. **Using print() instead of logging** — `print()` statements bypass the logging framework's filtering, formatting, and routing capabilities. Always use `logger.info()` etc.
3. **Setting log level too low in production** — DEBUG-level logging generates enormous volumes that can fill disks and degrade performance. Use INFO or WARNING in production.

## Best Practices

1. **Use structured (JSON) logging** — JSON logs are parseable by log aggregation tools, enabling powerful search and alerting.
2. **Set up Sentry for error tracking** — It automatically captures exceptions with context, eliminating the need to grep through logs for error details.
3. **Implement health check endpoints** — Health checks let load balancers and monitoring systems verify your application is working without relying solely on logs.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Django production logging with JSON formatter, file rotation, and Sentry integration**

```python
# settings/production.py

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'json': {
            '()': 'pythonjsonlogger.jsonlogger.JsonFormatter',
            'format': '%(asctime)s %(levelname)s %(name)s %(message)s',
        },
    },
    
    'filters': {
        'require_debug_false': {
            '()': 'django.utils.log.RequireDebugFalse',
# ...
```


## Resources

- [Django Logging](https://docs.djangoproject.com/en/6.0/topics/logging/) — Django logging documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*