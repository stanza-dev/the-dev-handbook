---
source_course: "django-deployment"
source_lesson: "django-deployment-process-management"
---

# Process Management

## Introduction

Process managers ensure your application starts on boot, restarts on crash, and can be gracefully reloaded during deployments.

## Key Concepts

**Systemd**: Modern Linux service manager.

**Graceful Restart**: Update without dropping connections.

**Socket Activation**: Start service on first request.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Deep Dive

### Systemd Service

```ini
# /etc/systemd/system/django.service
[Unit]
Description=Django
After=network.target

[Service]
User=www-data
WorkingDirectory=/var/www/myproject
EnvironmentFile=/var/www/myproject/.env
ExecStart=/var/www/myproject/venv/bin/gunicorn \
    --workers 4 \
    --bind unix:/run/gunicorn.sock \
    myproject.wsgi:application
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### Service Commands

```bash
sudo systemctl daemon-reload
sudo systemctl enable django
sudo systemctl start django
sudo systemctl reload django  # Graceful
```

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Run as non-root**: Use dedicated user like www-data.
2. **Auto-restart on failure**: Set Restart=on-failure.
3. **Use socket files**: More secure than TCP.

## Summary

Systemd manages your application lifecycle. Use socket files, run as non-root, and enable auto-restart for reliability.

## Code Examples

**Systemd service file for managing Django/Gunicorn process**

```python
# /etc/systemd/system/django.service
[Unit]
Description=Django
After=network.target

[Service]
User=www-data
WorkingDirectory=/var/www/myproject
EnvironmentFile=/var/www/myproject/.env
ExecStart=/var/www/myproject/venv/bin/gunicorn \
    --workers 4 \
    --bind unix:/run/gunicorn.sock \
    myproject.wsgi:application
Restart=on-failure

[Install]
WantedBy=multi-user.target
```


## Resources

- [Systemd Services](https://www.freedesktop.org/software/systemd/man/systemd.service.html) — Systemd documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*