---
source_course: "django-deployment"
source_lesson: "django-deployment-docker-best-practices"
---

# Docker Best Practices

## Introduction

Following Docker best practices ensures secure, efficient, and reproducible containers.

## Key Practices

### Run as Non-Root User

```dockerfile
RUN adduser --disabled-password appuser
RUN chown -R appuser:appuser /app
USER appuser
```

### Use .dockerignore

```
.git
.env*
*.pyc
__pycache__
media/*
staticfiles/*
```

### Pin Dependencies

```dockerfile
FROM python:3.11.6-slim  # Specific version
RUN pip install django==6.0.1  # Pinned versions
```

### Minimize Layers

```dockerfile
# Bad: Multiple RUN commands
RUN apt-get update
RUN apt-get install -y gcc
RUN apt-get clean

# Good: Combined
RUN apt-get update && \
    apt-get install -y gcc && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Environment Variables

```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
```

## Best Practices

1. **Non-root user**: Security best practice.
2. **Multi-stage builds**: Smaller, more secure images.
3. **Pin versions**: Reproducible builds.
4. **Use .dockerignore**: Faster builds, smaller context.

## Summary

Build secure Docker images by running as non-root, using multi-stage builds, pinning versions, and minimizing layers. Always use .dockerignore to exclude unnecessary files.

## Resources

- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) — Docker best practices

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*