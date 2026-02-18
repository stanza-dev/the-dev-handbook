---
source_course: "django-deployment"
source_lesson: "django-deployment-logging-configuration"
---

# Production Logging

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

## Resources

- [Django Logging](https://docs.djangoproject.com/en/6.0/topics/logging/) — Django logging documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*