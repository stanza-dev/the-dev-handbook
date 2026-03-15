---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-manual-setup"
---

# Manual Setup

## Introduction

While the automated onboard command works for most cases, understanding the manual setup process gives you full control over each step. This is useful when customizing the environment, debugging issues, or integrating Paperclip into an existing development workflow.

## Key Concepts

- **git clone**: Fetches the Paperclip source code from GitHub.
- **pnpm dev**: Starts the development server with file watching for live reloading.
- **pnpm dev:once**: Runs the server once without file watching, useful for production-like local testing.
- **pnpm dev:server**: Starts only the API server without the web UI.
- **.env.example**: Template file containing all available environment variables with default values.

## Real World Context

In a team environment, you may need to point Paperclip at an existing PostgreSQL instance, use a custom port, or skip certain setup steps that conflict with your infrastructure. Manual setup lets you configure each piece independently, which is essential for teams that manage their own databases or use container orchestration.

## Deep Dive

Start by cloning the repository and installing dependencies.

```bash
git clone https://github.com/paperclipai/paperclip.git
cd paperclip
pnpm install
```

Before starting the server, copy the environment template and adjust any values you need.

```bash
cp .env.example .env
```

The `.env` file contains configuration for the database connection, server port, authentication settings, and storage paths. For local development with defaults, no changes are necessary.

Paperclip provides three development modes, each suited to a different workflow.

```bash
# Full dev mode with file watching (recommended for development)
pnpm dev

# Single run without file watching
pnpm dev:once

# API server only, no web UI
pnpm dev:server
```

The `pnpm dev` command starts both the API and web UI with automatic reloading when source files change. This is the best choice for active development. The `pnpm dev:once` variant is useful when you want a stable server without restarts, such as when running integration tests. The `pnpm dev:server` mode is for headless environments or when you are developing a custom frontend.

For production builds, use the build command.

```bash
pnpm build
```

This compiles TypeScript, bundles assets, and produces optimized output ready for deployment.

## Common Pitfalls

1. **Forgetting to copy .env.example** — Running without a `.env` file uses hardcoded defaults, which may not match your environment. Always start from the template.
2. **Using `pnpm dev` in production** — The dev mode includes file watching, source maps, and verbose logging that hurt performance. Use `pnpm build` followed by a production start command instead.

## Best Practices

1. **Use `pnpm dev:server` when building a custom UI** — If you are developing your own frontend against the Paperclip API, skip the built-in web UI to avoid port conflicts and reduce resource usage.
2. **Keep .env out of version control** — The `.env` file may contain secrets. Ensure it is listed in `.gitignore` and share configuration via `.env.example` with placeholder values.

## Summary

- Clone the repo with `git clone` and install with `pnpm install`.
- Copy `.env.example` to `.env` for configuration.
- `pnpm dev` provides full development mode with file watching.
- `pnpm dev:once` and `pnpm dev:server` offer alternative startup modes.
- Use `pnpm build` for production-ready output.

## Code Examples

**Step-by-step manual Paperclip setup from source**

```bash
git clone https://github.com/paperclipai/paperclip.git
cd paperclip
pnpm install
cp .env.example .env
pnpm dev
```


## Resources

- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Source code and setup instructions

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*