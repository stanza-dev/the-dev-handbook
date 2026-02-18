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

## Best Practices

1. **Run as non-root**: Use dedicated user like www-data.
2. **Auto-restart on failure**: Set Restart=on-failure.
3. **Use socket files**: More secure than TCP.

## Summary

Systemd manages your application lifecycle. Use socket files, run as non-root, and enable auto-restart for reliability.

## Resources

- [Systemd Services](https://www.freedesktop.org/software/systemd/man/systemd.service.html) — Systemd documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*