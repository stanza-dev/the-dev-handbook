---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-database-storage"
---

# Database & Storage Configuration

## Introduction

Paperclip supports two database backends and two storage backends. Understanding when to switch from the defaults is essential for production deployments. This lesson covers PGlite vs PostgreSQL for the database and local filesystem vs S3-compatible storage for file assets.

## Key Concepts

- **PGlite**: Embedded PostgreSQL that runs inside the Node.js process. Zero external dependencies, stores data in a local directory.
- **PostgreSQL 17**: External database server for production deployments with multi-user access, backups, and replication.
- **Local file storage**: Default storage backend that writes files to the local filesystem.
- **S3-compatible storage**: Cloud storage backend for production, supporting AWS S3, MinIO, Cloudflare R2, and similar services.

## Real World Context

A team starts with PGlite during evaluation and early development. As they move toward production, they need database backups, point-in-time recovery, connection pooling, and the ability to run multiple Paperclip instances against the same data. Switching to external PostgreSQL 17 provides all of these capabilities. Similarly, local file storage works for a single instance but fails when scaling horizontally — S3-compatible storage lets multiple instances share the same file assets.

## Deep Dive

PGlite is the default database and requires no configuration. It stores data in the directory specified by `--data-dir` or the default `~/.paperclip/` location.

When you are ready for production, switch to external PostgreSQL 17.

```bash
# Set the database URL in .env
DATABASE_URL=postgresql://user:password@localhost:5432/paperclip
```

After changing the database URL, generate the Prisma client and run migrations.

```bash
# Generate Prisma client for the new database
pnpm db:generate

# Run migrations to create tables
pnpm db:migrate
```

The `db:generate` command creates the TypeScript client that matches your schema. The `db:migrate` command applies all pending migrations to create or update tables in the target database. Always run both commands after switching databases.

For storage, the default is the local filesystem. Configure S3-compatible storage via environment variables.

```bash
# S3-compatible storage configuration in .env
STORAGE_TYPE=s3
S3_BUCKET=paperclip-assets
S3_REGION=us-east-1
S3_ACCESS_KEY_ID=AKIA...
S3_SECRET_ACCESS_KEY=...
S3_ENDPOINT=https://s3.amazonaws.com  # Or MinIO/R2 endpoint
```

The storage backend is used for file attachments, agent work artifacts, and exported configurations. Switching to S3 is recommended when running multiple Paperclip instances or when you need durable cloud storage with built-in redundancy.

Decide when to switch based on these criteria: use PGlite for local development and single-user setups; switch to PostgreSQL 17 for production, multi-user deployments, or when you need proper backups. Use local storage for development; switch to S3 when running multiple instances or deploying to cloud infrastructure.

## Common Pitfalls

1. **Forgetting to run db:migrate after switching databases** — The new PostgreSQL database has no tables. Without running migrations, Paperclip will crash on startup with "relation does not exist" errors.
2. **Using PGlite in production with multiple users** — PGlite is single-process and not designed for concurrent multi-user access. It works for a solo developer but will produce data corruption under concurrent writes from multiple users.

## Best Practices

1. **Switch to PostgreSQL before going multi-user** — Do not wait until you experience PGlite issues. Migrate proactively when your second user joins.
2. **Use the same PostgreSQL version (17) as Paperclip tests against** — While older versions may work, using version 17 ensures compatibility with all Prisma migrations and features.

## Summary

- PGlite is the default embedded database — great for development, not for production.
- PostgreSQL 17 is the production database — set `DATABASE_URL` and run `pnpm db:generate` and `pnpm db:migrate`.
- Local filesystem is the default storage — works for single instances.
- S3-compatible storage is for production — supports multi-instance deployments.
- Always run migrations after switching databases.

## Code Examples

**Switching from PGlite to external PostgreSQL 17**

```bash
# Switch to external PostgreSQL
# In .env:
# DATABASE_URL=postgresql://user:pass@localhost:5432/paperclip

# Generate client and run migrations
pnpm db:generate
pnpm db:migrate
```


## Resources

- [Paperclip Database Configuration](https://paperclip.ing/docs) — Database and storage backend configuration guide

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*