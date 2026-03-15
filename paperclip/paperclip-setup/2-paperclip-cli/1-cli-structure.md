---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-cli-structure"
---

# CLI Structure & Global Flags

## Introduction

The Paperclip CLI is the primary interface for managing your Paperclip installation beyond the web UI. It provides commands for setup, configuration, diagnostics, and full control-plane operations. Understanding its structure and global flags is essential for efficient workflows.

## Key Concepts

- **Setup Commands**: Commands for initial configuration — `onboard`, `configure`, `doctor`, and `run`.
- **Control-Plane Commands**: Commands for managing resources — `issue`, `agent`, `approval`, and `company`.
- **Global Flags**: Flags available on every command — `--data-dir`, `--api-base`, `--api-key`, `--json`, `--company-id`.
- **--json flag**: Outputs results in machine-readable JSON format for scripting and automation.
- **--context and --profile**: Select which saved configuration profile to use for the current command.

## Real World Context

As your Paperclip usage grows from a single local instance to multiple environments (development, staging, production), the CLI becomes the fastest way to manage agents and issues across all of them. Global flags let you target different instances from a single terminal, and the `--json` flag enables integration with shell scripts, CI pipelines, and monitoring tools.

## Deep Dive

The CLI is invoked via `pnpm paperclipai` followed by a command and optional flags.

```bash
# View all available commands and flags
pnpm paperclipai --help
```

The output shows two categories of commands. Setup commands handle installation and maintenance. Control-plane commands manage the resources within a running Paperclip instance.

Global flags can be passed to any command to override default behavior.

```bash
# Override the data directory
pnpm paperclipai --data-dir /custom/path issue list

# Target a remote Paperclip instance
pnpm paperclipai --api-base http://staging.example.com:3100 agent list

# Authenticate with an API key
pnpm paperclipai --api-key pk_live_abc123 company list

# Output as JSON for scripting
pnpm paperclipai --json issue list
```

The `--data-dir` flag controls where Paperclip stores its local data, including the PGlite database and configuration files. The `--api-base` flag points the CLI at a specific Paperclip server, which is essential when managing remote or multi-environment deployments.

The `--company-id` flag scopes commands to a specific company. Most control-plane commands require this flag unless a default company is set in the active context profile.

```bash
# Scope to a specific company
pnpm paperclipai --company-id comp_abc123 issue list

# Use a named profile
pnpm paperclipai --profile staging issue list
```

The `--context` and `--profile` flags select a saved configuration profile, which bundles together an API base URL, company ID, and API key. This avoids repeating flags on every command.

## Common Pitfalls

1. **Forgetting --company-id on control-plane commands** — Commands like `issue list` or `agent list` require a company scope. Without it, the CLI returns an error or empty results. Set a default via context profiles to avoid this.
2. **Mixing up --api-key and --api-base** — The `--api-key` authenticates you to the server; `--api-base` tells the CLI where the server is. Using one without the other when targeting a remote authenticated instance will fail.

## Best Practices

1. **Use --json for automation** — When integrating Paperclip into scripts or CI, always pass `--json` to get structured output that can be parsed with `jq` or similar tools.
2. **Set up context profiles for each environment** — Instead of passing `--api-base` and `--company-id` on every command, configure named profiles for local, staging, and production.

## Summary

- The CLI has two command categories: setup (`onboard`, `configure`, `doctor`, `run`) and control-plane (`issue`, `agent`, `approval`, `company`).
- Global flags (`--data-dir`, `--api-base`, `--api-key`, `--json`, `--company-id`) work on every command.
- Use `--json` for machine-readable output in scripts.
- Use `--profile` to switch between saved configurations for different environments.

## Code Examples

**Using global flags to query a remote Paperclip instance**

```bash
# List issues as JSON from a remote instance
pnpm paperclipai --api-base http://staging:3100 \
  --company-id comp_abc123 --json issue list
```


## Resources

- [Paperclip CLI Reference](https://paperclip.ing/docs) — Complete CLI command and flag reference

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*