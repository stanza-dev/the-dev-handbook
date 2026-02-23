---
source_course: "django-deployment"
source_lesson: "django-deployment-compose-production"
---

# Docker Compose for Production

## Introduction

Docker Compose orchestrates multi-container Django deployments, defining your web server, database, cache, and worker processes in a single configuration file. Production Compose files add health checks, resource limits, and secrets management.

## Production Compose File

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    image: myregistry/myproject:${TAG:-latest}
    restart: always
    env_file:
      - .env.production
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
    healthcheck:
      test: curl -f http://localhost:8000/health/
      interval: 30s
      timeout: 10s
      retries: 3

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - static_volume:/var/www/static:ro
    depends_on:
      - web

  db:
    image: postgres:15
    restart: always
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

volumes:
  postgres_data:
  static_volume:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Deployment Commands

```bash
# Build and push
docker-compose -f docker-compose.prod.yml build
docker-compose -f docker-compose.prod.yml push

# Deploy
docker-compose -f docker-compose.prod.yml up -d

# Rolling update
docker-compose -f docker-compose.prod.yml up -d --no-deps web
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

1. **Separate files**: dev vs production compose files.
2. **Use secrets**: Don't put passwords in compose files.
3. **Health checks**: Enable automatic recovery.
4. **Resource limits**: Prevent runaway containers.

## Summary

Use separate compose files for production with health checks, secrets, resource limits, and restart policies. Consider Kubernetes for larger deployments.

## Code Examples

**Production Docker Compose with health checks, secrets, and resource limits**

```bash
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    image: myregistry/myproject:${TAG:-latest}
    restart: always
    env_file:
      - .env.production
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
    healthcheck:
      test: curl -f http://localhost:8000/health/
      interval: 30s
      timeout: 10s
      retries: 3

# ...
```


## Resources

- [Compose in Production](https://docs.docker.com/compose/production/) — Docker Compose production guide

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*