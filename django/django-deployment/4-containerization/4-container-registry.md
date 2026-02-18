---
source_course: "django-deployment"
source_lesson: "django-deployment-container-registry"
---

# Container Registries

## Introduction

Container registries store and distribute Docker images. You need a registry to deploy containers to production.

## Registry Options

**Docker Hub**: Public and private repositories.

**AWS ECR**: Integrated with AWS services.

**Google GCR**: Integrated with Google Cloud.

**GitHub Container Registry**: Integrated with GitHub Actions.

## Pushing to Registry

```bash
# Login
docker login myregistry.com

# Tag image
docker tag myproject:latest myregistry.com/myproject:v1.0.0

# Push
docker push myregistry.com/myproject:v1.0.0
```

## CI/CD Integration

```yaml
# GitHub Actions
- name: Build and push
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

## Best Practices

1. **Use tags**: Never rely on 'latest' in production.
2. **Immutable tags**: Use git SHA or semantic versions.
3. **Scan for vulnerabilities**: Most registries offer scanning.

## Summary

Choose a registry based on your infrastructure. Use immutable tags (git SHA or semver), enable vulnerability scanning, and integrate with CI/CD for automated builds.

## Resources

- [Docker Registry](https://docs.docker.com/registry/) — Docker Registry documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*