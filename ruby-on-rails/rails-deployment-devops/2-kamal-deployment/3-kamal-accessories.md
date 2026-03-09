---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-kamal-accessories"
---

# Kamal Accessories and Secrets

## Introduction
Kamal manages not just your application containers but also supporting services like databases, caches, and search engines through accessories. Combined with secure secrets management, Kamal provides a complete deployment solution.

## Key Concepts
- **Accessory**: A supporting service (PostgreSQL, Redis, etc.) that Kamal deploys and manages alongside your application.
- **Secrets**: Sensitive values stored in `.kamal/secrets` and injected as environment variables at runtime — never committed to version control.
- **Destinations**: Named deployment targets (staging, production) with different configurations.

## Real World Context
Instead of manually installing PostgreSQL on your server and managing backups, Kamal accessories run the database as a Docker container with persistent volumes. One `kamal accessory boot db` command sets everything up.

## Deep Dive

### Accessories Configuration

```yaml
# config/deploy.yml
accessories:
  db:
    image: postgres:17
    host: 192.168.1.4
    port: 5432
    env:
      secret:
        - POSTGRES_PASSWORD
    volumes:
      - /data/postgres:/var/lib/postgresql/data

  search:
    image: elasticsearch:8.12.0
    host: 192.168.1.4
    port: 9200
    env:
      clear:
        discovery.type: single-node
        xpack.security.enabled: "false"
    volumes:
      - /data/elasticsearch:/usr/share/elasticsearch/data
```

### Managing Accessories

```bash
kamal accessory boot db        # Start the database
kamal accessory reboot db      # Restart the database
kamal accessory logs db        # View database logs
kamal accessory exec db 'psql' # Open psql shell
```

### Secrets Management

```bash
# .kamal/secrets (never committed to git)
KAMAL_REGISTRY_PASSWORD=ghp_xxxxxxxxxxxx
RAILS_MASTER_KEY=abc123def456
DATABASE_URL=postgres://app:secret@db:5432/myapp_production
POSTGRES_PASSWORD=very_secure_password
```

```yaml
# config/deploy.yml
env:
  clear:
    RAILS_ENV: production
    SOLID_QUEUE_IN_PUMA: true
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL
```

### Destinations for Multi-Environment

```yaml
# config/deploy.staging.yml
servers:
  web:
    hosts:
      - staging.example.com

proxy:
  host: staging.myapp.com
```

```bash
kamal deploy --destination staging
kamal deploy --destination production
```

## Common Pitfalls
1. **Committing secrets to git** — Add `.kamal/secrets` to `.gitignore`. These contain passwords and API keys that must never be in version control.
2. **Not using persistent volumes for databases** — Without a volume mount, database data is lost when the container restarts.

## Best Practices
1. **Use separate hosts for accessories in production** — Running the database on the same server as the app means a server failure loses both.
2. **Use destinations for staging** — Maintain separate deploy configurations so staging and production can't be confused.

## Summary
- Accessories manage supporting services like databases alongside your app.
- Secrets are stored in `.kamal/secrets` and injected as environment variables.
- Use persistent volumes for any stateful service (databases, search).
- Destinations allow separate configurations for staging and production.

## Code Examples

**PostgreSQL accessory configuration — Kamal manages the database container with persistent storage**

```yaml
# config/deploy.yml — accessories section
accessories:
  db:
    image: postgres:17
    host: 192.168.1.4
    port: 5432
    env:
      secret:
        - POSTGRES_PASSWORD
    volumes:
      - /data/postgres:/var/lib/postgresql/data

# Manage with:
# kamal accessory boot db
# kamal accessory logs db
# kamal accessory exec db 'psql -U postgres'
```


## Resources

- [Kamal Accessories Documentation](https://kamal-deploy.org/docs/configuration/accessories/) — Official Kamal documentation on configuring and managing accessories

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*