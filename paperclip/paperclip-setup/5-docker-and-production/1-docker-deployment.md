---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-docker-deployment"
---

# Docker Deployment

## Introduction

Docker provides a consistent, reproducible way to deploy Paperclip without worrying about host system dependencies. Whether you are running a quick local test or deploying to a cloud server, Docker containers ensure the same behavior everywhere. This lesson covers the Docker run command, docker-compose setup, and volume persistence.

## Key Concepts

- **Docker container**: An isolated runtime that packages Paperclip with all its dependencies.
- **Port 3100**: The default port Paperclip listens on inside the container, mapped to the host.
- **PAPERCLIP_HOME**: Environment variable that sets the data directory inside the container.
- **Volume mount**: Persists Paperclip's data directory outside the container so data survives container restarts.

## Real World Context

A DevOps engineer needs to deploy Paperclip on a cloud VM for the team. Installing Node.js, pnpm, and managing system dependencies on a production server is fragile and hard to reproduce. A single `docker run` command with a volume mount gives them a running instance in seconds, with data persistence and easy upgrades — just pull the new image and restart.

## Deep Dive

The quickest way to run Paperclip in Docker is a single `docker run` command.

```bash
docker run --name paperclip \
  -p 3100:3100 \
  -e HOST=0.0.0.0 \
  -e PAPERCLIP_HOME=/paperclip \
  -v "$(pwd)/data:/paperclip" \
  paperclip-local
```

This command maps port 3100 from the container to the host, sets the `HOST` environment variable to `0.0.0.0` so the server accepts connections from outside the container, configures `PAPERCLIP_HOME` to `/paperclip` inside the container, and mounts a local `data` directory to persist all Paperclip data including the PGlite database.

The volume mount at `-v "$(pwd)/data:/paperclip"` is critical. Without it, all data is lost when the container stops. The mounted directory stores the PGlite database, configuration files, and any local file storage assets.

For more complex deployments, use docker-compose to orchestrate Paperclip with an external PostgreSQL database.

```yaml
# docker-compose.yml
version: '3.8'
services:
  paperclip:
    image: paperclip-local
    ports:
      - "3100:3100"
    environment:
      HOST: 0.0.0.0
      PAPERCLIP_HOME: /paperclip
      DATABASE_URL: postgresql://paperclip:secret@db:5432/paperclip
    volumes:
      - paperclip-data:/paperclip
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:17
    environment:
      POSTGRES_USER: paperclip
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: paperclip
    volumes:
      - pg-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U paperclip"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  paperclip-data:
  pg-data:
```

This compose file starts both Paperclip and PostgreSQL 17, with a health check ensuring the database is ready before Paperclip starts. Named volumes persist data across container restarts and upgrades.

Start the stack with a single command.

```bash
docker compose up -d
```

The `-d` flag runs containers in the background. Access the UI at `http://localhost:3100`.

## Common Pitfalls

1. **Forgetting the volume mount** — Without `-v`, all data lives inside the container and is destroyed when the container is removed. Always mount a volume for PAPERCLIP_HOME.
2. **Not setting HOST=0.0.0.0** — By default, Paperclip binds to localhost. Inside a Docker container, localhost means the container itself, not the host machine. Set `HOST=0.0.0.0` to accept connections forwarded from the Docker port mapping.

## Best Practices

1. **Use named volumes in production** — Named Docker volumes (`paperclip-data:`) are managed by Docker and survive `docker compose down`. Bind mounts (`./data:/paperclip`) are easier for development but riskier in production.
2. **Add health checks for dependent services** — The `depends_on` with `condition: service_healthy` ensures Paperclip does not start before PostgreSQL is ready, preventing startup crashes.

## Summary

- `docker run` with port mapping, HOST=0.0.0.0, and a volume mount is the quickest Docker deployment.
- PAPERCLIP_HOME sets the data directory inside the container.
- Use docker-compose for multi-service deployments with external PostgreSQL.
- Always mount volumes to persist data across container restarts.
- Set HOST=0.0.0.0 so the containerized server accepts external connections.

## Code Examples

**Running Paperclip in Docker with data persistence**

```bash
docker run --name paperclip \
  -p 3100:3100 \
  -e HOST=0.0.0.0 \
  -e PAPERCLIP_HOME=/paperclip \
  -v "$(pwd)/data:/paperclip" \
  paperclip-local
```


## Resources

- [Paperclip Docker Guide](https://paperclip.ing/docs) — Docker deployment instructions and compose examples

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*