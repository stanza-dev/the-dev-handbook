---
source_course: "django-deployment"
source_lesson: "django-deployment-alerting"
---

# Alerting Strategies

## Introduction

Alerts notify you when something needs attention. Good alerting catches issues early without causing alert fatigue.

## What to Alert On

**Symptoms**: High error rate, high latency, service down.

**NOT Causes**: High CPU alone doesn't mean there's a problem.

## Prometheus Alerting

```yaml
groups:
- name: django
  rules:
  - alert: HighErrorRate
    expr: rate(http_responses_total{status="5xx"}[5m]) > 0.1
    for: 5m
    annotations:
      summary: High error rate detected
```

## Sentry Alerts

```python
import sentry_sdk
sentry_sdk.init(
    dsn=os.environ['SENTRY_DSN'],
    traces_sample_rate=0.1,
)
# Sentry auto-alerts on new errors
```

## Key Concepts

The key terms and concepts for this topic are introduced in the Deep Dive section below.


## Deep Dive

See the detailed technical content and code examples throughout this lesson.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Alert on symptoms**: User-facing impact.
2. **Avoid noise**: Only alert on actionable issues.
3. **Write runbooks**: Document responses.
4. **Test alerts**: Verify they fire correctly.

## Summary

Alert on symptoms that affect users, not just causes. Use Prometheus AlertManager or Sentry. Avoid alert fatigue by keeping alerts actionable.

## Code Examples

**Prometheus alerting rule for high error rate detection**

```yaml
import sentry_sdk
sentry_sdk.init(
    dsn=os.environ['SENTRY_DSN'],
    traces_sample_rate=0.1,
)
# Sentry auto-alerts on new errors
```


## Resources

- [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) — Prometheus Alertmanager

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*