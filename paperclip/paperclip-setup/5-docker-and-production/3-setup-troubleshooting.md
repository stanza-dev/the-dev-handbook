---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-troubleshooting"
---

# Troubleshooting & Maintenance

## Introduction

Even well-configured deployments encounter issues. Paperclip provides built-in diagnostic tools and self-repair capabilities to help you identify and resolve problems quickly. This lesson covers the doctor command, common startup issues, log analysis, and the decision between restarting and reconfiguring.

## Key Concepts

- **pnpm paperclipai doctor**: Diagnostic command that checks system health and reports issues.
- **pnpm paperclipai run**: Includes auto-repair that fixes common problems before starting.
- **db:migrate**: Applies pending database migrations, often needed after upgrades.
- **Logs**: Server output that reveals startup errors, connection failures, and runtime issues.

## Real World Context

A developer upgrades Paperclip to a new version and the server fails to start. Instead of searching through error logs or asking for help, they run `pnpm paperclipai doctor` which immediately identifies that three database migrations are pending. Running `pnpm db:migrate` resolves the issue. The doctor command turns a potentially hours-long debugging session into a two-minute fix.

## Deep Dive

The `doctor` command is the first tool to reach for when something goes wrong.

```bash
pnpm paperclipai doctor
```

Doctor checks several aspects of the system: Node.js and pnpm versions, database connectivity and migration status, Prisma client generation status, environment variable configuration, and port availability. It reports each check as passed, warning, or failed with specific remediation steps.

The `run` command includes lighter self-repair logic that runs automatically before starting the server.

```bash
# Auto-repair + start
pnpm paperclipai run
```

If `run` detects a stale Prisma client, it regenerates it. If migrations are pending, it runs them. If the PGlite database is corrupted, it attempts recovery. This makes `run` safer than `pnpm dev` for recovering from issues.

Common startup issues and their solutions follow a pattern.

```bash
# Issue: "relation does not exist" errors
# Cause: Pending database migrations after upgrade
# Fix:
pnpm db:migrate

# Issue: "Cannot find module @prisma/client"
# Cause: Prisma client not generated or outdated
# Fix:
pnpm db:generate

# Issue: "EADDRINUSE: port 3100"
# Cause: Another process is using the port
# Fix:
lsof -i :3100  # Find the process
kill -9 <PID>  # Terminate it

# Issue: "ECONNREFUSED" to database
# Cause: PostgreSQL is not running or URL is wrong
# Fix:
docker compose up -d db  # If using Docker
# Or verify DATABASE_URL in .env
```

Check logs for detailed error information when the doctor command does not identify the issue.

```bash
# View recent server logs
pnpm paperclipai run 2>&1 | tail -50

# For Docker deployments
docker logs paperclip --tail 50
```

When deciding between restart and reconfigure, follow this guideline: if the issue started after an upgrade, run `doctor` then `db:migrate`. If the issue started after an environment change (new server, new database), run `configure` to update settings. If the issue is intermittent, restart first and check logs if it persists.

## Common Pitfalls

1. **Ignoring doctor output** — Running `doctor` and skipping the warnings often leads to the same issue recurring. Address all warnings, not just errors.
2. **Killing and restarting without checking logs** — Blindly restarting masks the root cause. Always check logs or run doctor before restarting to understand what went wrong.

## Best Practices

1. **Run doctor after every upgrade** — New versions may require migrations, updated Prisma clients, or new environment variables. Doctor catches all of these.
2. **Keep a troubleshooting log** — Document issues you encounter and their solutions. This becomes invaluable for team onboarding and recurring problems.

## Summary

- `pnpm paperclipai doctor` diagnoses system health with specific remediation steps.
- `pnpm paperclipai run` includes auto-repair for common issues.
- Common startup fixes: `db:migrate` for missing tables, `db:generate` for stale Prisma client.
- Check logs before restarting to identify root causes.
- Run doctor after every upgrade to catch required migrations and config changes.

## Code Examples

**Diagnostic and repair commands for Paperclip maintenance**

```bash
# Diagnose issues
pnpm paperclipai doctor

# Fix common post-upgrade issues
pnpm db:migrate
pnpm db:generate

# Start with auto-repair
pnpm paperclipai run
```


## Resources

- [Paperclip Troubleshooting](https://paperclip.ing/docs) — Common issues and their solutions

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*