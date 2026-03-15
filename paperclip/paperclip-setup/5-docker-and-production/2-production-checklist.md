---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-production-checklist"
---

# Production Checklist

## Introduction

Moving Paperclip from a local development setup to a production deployment requires changes across several areas: database, authentication, networking, monitoring, and backups. This lesson provides a systematic checklist to ensure nothing is missed.

## Key Concepts

- **Production database**: Switching from PGlite to PostgreSQL 17 for reliability and multi-user support.
- **Base URL configuration**: Setting explicit URLs for authentication redirects and CORS.
- **Better Auth**: Configuring the authentication system for multi-user access.
- **Tailscale**: A recommended solution for private team access without exposing Paperclip to the internet.
- **Health checks**: Automated verification that the service is running correctly.

## Real World Context

A startup has been running Paperclip locally during their two-week evaluation. They are ready to deploy it for the whole engineering team of twelve people. The CTO needs confidence that the deployment is reliable, data is backed up, and access is properly controlled. A production checklist turns this from a guessing game into a systematic process.

## Deep Dive

Work through this checklist before considering a deployment production-ready.

First, switch from PGlite to external PostgreSQL 17.

```bash
# Set DATABASE_URL in .env or environment
DATABASE_URL=postgresql://paperclip:strong-password@db-host:5432/paperclip

# Generate client and migrate
pnpm db:generate
pnpm db:migrate
```

PGlite is single-process and not designed for concurrent multi-user access. PostgreSQL 17 provides connection pooling, ACID transactions under concurrency, point-in-time recovery, and replication.

Second, set explicit base URLs.

```bash
# For Auth + Private (Tailscale)
BASE_URL=http://100.64.1.50:3100

# For Auth + Public (internet-facing)
BASE_URL=https://paperclip.mycompany.com
```

The base URL is used for authentication redirects, CORS allowed origins, and cookie domains. An incorrect base URL breaks login flows silently.

Third, configure Better Auth for user management.

```bash
# Configure authentication
pnpm paperclipai configure --section auth
# Select authenticated mode (private or public)
# Set up initial admin credentials
```

Fourth, set up private networking with Tailscale if you do not need public internet access.

```bash
# Install Tailscale on the server
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Paperclip is now accessible at the Tailscale IP
# Only devices on your Tailscale network can reach it
```

Tailscale creates an encrypted mesh network with zero firewall configuration. Team members install Tailscale on their devices and can access Paperclip as if it were on their local network.

Fifth, implement health checks for monitoring.

```bash
# Basic health check
curl -f http://localhost:3100/api/health || echo "Paperclip is down"

# Add to your monitoring system (e.g., cron job)
*/5 * * * * curl -sf http://localhost:3100/api/health > /dev/null || alert "Paperclip down"
```

Sixth, establish a backup strategy. With PostgreSQL, use `pg_dump` for logical backups or continuous archiving for point-in-time recovery.

```bash
# Daily database backup
pg_dump $DATABASE_URL > backup-$(date +%Y%m%d).sql

# For S3 storage, backup the bucket separately
aws s3 sync s3://paperclip-assets ./backup-assets/
```

## Common Pitfalls

1. **Skipping the database migration** — Running PGlite in production with multiple users leads to data corruption under concurrent writes. This is the single most important checklist item.
2. **Using HTTP instead of HTTPS for Auth + Public** — Authentication tokens sent over unencrypted HTTP can be intercepted. Always use HTTPS for internet-facing deployments.

## Best Practices

1. **Automate the health check** — A manual health check is a health check that does not happen. Integrate with your monitoring system from day one.
2. **Test backup restoration** — A backup you have never restored is not a backup. Periodically restore to a test environment to verify the process works.

## Summary

- Switch PGlite to PostgreSQL 17 for multi-user reliability.
- Set explicit base URLs for auth redirects and CORS.
- Configure Better Auth for user management.
- Use Tailscale for private team access without internet exposure.
- Implement health checks and integrate with monitoring.
- Establish and test backup procedures for both database and storage.

## Code Examples

**Essential production setup commands**

```bash
# Production checklist key commands
export DATABASE_URL=postgresql://user:pass@db:5432/paperclip
pnpm db:generate && pnpm db:migrate
pnpm paperclipai configure --section auth
curl -f http://localhost:3100/api/health
```


## Resources

- [Paperclip Production Guide](https://paperclip.ing/docs) — Production deployment best practices and configuration

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*