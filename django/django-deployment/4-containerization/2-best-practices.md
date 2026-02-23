---
source_course: "django-deployment"
source_lesson: "django-deployment-docker-best-practices"
---

# Docker Best Practices

## Introduction

Docker best practices focus on security, reproducibility, and image size. Running as a non-root user, pinning base image versions, minimizing layers, and using `.dockerignore` are essential for production-ready containers.

## Key Practices

### Run as Non-Root User

```dockerfile
RUN adduser --disabled-password appuser
RUN chown -R appuser:appuser /app
USER appuser
```

### Use .dockerignore

```dockerfile
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

1. **Non-root user**: Security best practice.
2. **Multi-stage builds**: Smaller, more secure images.
3. **Pin versions**: Reproducible builds.
4. **Use .dockerignore**: Faster builds, smaller context.

## Summary

Build secure Docker images by running as non-root, using multi-stage builds, pinning versions, and minimizing layers. Always use .dockerignore to exclude unnecessary files.

## Code Examples

**Docker best practices — non-root user, pinned versions, minimized layers**

```bash
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


## Resources

- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) — Docker best practices

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*