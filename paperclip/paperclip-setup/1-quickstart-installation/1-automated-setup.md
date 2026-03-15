---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-automated-setup"
---

# Automated Setup

## Introduction

Paperclip offers a one-command automated setup that handles cloning, installing dependencies, configuring the environment, and launching the application. This is the fastest way to get a working Paperclip instance on your local machine.

## Key Concepts

- **npx paperclipai onboard**: The single command that bootstraps an entire Paperclip installation from scratch.
- **PGlite**: An embedded PostgreSQL-compatible database that eliminates the need for an external database server during local development.
- **Auto-repair**: The `pnpm paperclipai run` command can detect and fix common configuration issues automatically.

## Real World Context

When evaluating a new developer tool, the first experience matters. A complicated multi-step install process drives people away before they even see the product. Paperclip's one-command onboarding ensures that a developer can go from zero to a running AI agent platform in under two minutes, with no external dependencies beyond Node.js and pnpm.

## Deep Dive

Before running the automated setup, ensure you have the two prerequisites installed on your system.

```bash
# Check Node.js version (20+ required)
node --version

# Check pnpm version (9.15+ required)
pnpm --version
```

With prerequisites confirmed, the entire setup is a single command.

```bash
npx paperclipai onboard --yes
```

This command performs several steps behind the scenes: it clones the Paperclip repository, installs all dependencies via pnpm, generates the Prisma client, configures the embedded PGlite database, runs migrations, and launches both the API server and web UI. The `--yes` flag accepts all default options without interactive prompts.

Once the command completes, the Paperclip UI is available at `http://localhost:3100`. The API is served from the same port under the `/api` path.

If you need to restart Paperclip later, use the run command instead of re-running onboard.

```bash
pnpm paperclipai run
```

The `run` command includes auto-repair logic. If it detects missing migrations, a stale Prisma client, or corrupted configuration, it attempts to fix the issue before starting the server. This makes it the recommended way to launch Paperclip after the initial setup.

## Common Pitfalls

1. **Using an older Node.js version** — Paperclip requires Node.js 20 or higher. Versions 18 and below will fail during installation with cryptic dependency errors. Use `nvm install 20` to upgrade.
2. **Running with npm instead of pnpm** — The project uses pnpm workspaces. Running `npm install` will produce a broken dependency tree. Always use pnpm 9.15+.

## Best Practices

1. **Use the --yes flag for scripted installs** — When automating setup in CI or onboarding scripts, the `--yes` flag prevents the process from hanging on interactive prompts.
2. **Prefer `pnpm paperclipai run` for daily use** — After the initial onboard, use `run` instead of `onboard`. It starts faster and includes self-healing capabilities.

## Summary

- Prerequisites are Node.js 20+ and pnpm 9.15+.
- `npx paperclipai onboard --yes` handles the full bootstrap process.
- PGlite provides an embedded database with zero external dependencies.
- The UI launches at http://localhost:3100.
- Use `pnpm paperclipai run` for subsequent starts with auto-repair.

## Code Examples

**Automated Paperclip installation and launch commands**

```bash
# One-command setup
npx paperclipai onboard --yes

# Subsequent launches with auto-repair
pnpm paperclipai run
```


## Resources

- [Paperclip Quick Start](https://paperclip.ing/docs) — Official Paperclip documentation and quick start guide

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*