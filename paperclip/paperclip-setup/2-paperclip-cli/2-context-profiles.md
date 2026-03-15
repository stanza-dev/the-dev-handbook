---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-context-profiles"
---

# Context Profiles

## Introduction

Context profiles save your CLI configuration so you do not have to pass `--api-base`, `--company-id`, and `--api-key` on every command. Profiles are stored locally and can be switched instantly, making it easy to manage multiple Paperclip environments from one machine.

## Key Concepts

- **context set**: Creates or updates a profile with connection details.
- **context show**: Displays the currently active profile and its settings.
- **context list**: Shows all saved profiles.
- **context use**: Switches the active profile.
- **~/.paperclip/context.json**: The file where all profiles are persisted.

## Real World Context

A developer working on three projects has three Paperclip instances: a local one for personal experiments, a shared staging server for the team, and a production instance. Without profiles, every CLI command requires typing the full URL, company ID, and API key. With profiles, they type `pnpm paperclipai context use production` once and all subsequent commands target the right instance automatically.

## Deep Dive

Create a profile by setting connection details with the `context set` command.

```bash
# Set the API base URL for the current profile
pnpm paperclipai context set --api-base http://localhost:3100

# Set the company ID
pnpm paperclipai context set --company-id comp_abc123
```

You can combine multiple settings in a single command.

```bash
# Configure a complete profile
pnpm paperclipai context set \
  --api-base http://localhost:3100 \
  --company-id comp_abc123
```

For API keys, avoid passing the key directly on the command line where it would appear in shell history. Instead, reference an environment variable.

```bash
# Store API key securely via environment variable reference
export PAPERCLIP_API_KEY=pk_live_abc123
pnpm paperclipai context set --api-key env:PAPERCLIP_API_KEY
```

View the current profile configuration to verify your settings.

```bash
# Show active profile
pnpm paperclipai context show

# List all profiles
pnpm paperclipai context list
```

Switch between profiles when changing environments.

```bash
# Switch to a different profile
pnpm paperclipai context use staging
```

All profile data is persisted in `~/.paperclip/context.json`. This file contains profile names, API URLs, company IDs, and environment variable references (not the actual secret values). You can back up or share this file safely since it does not contain raw API keys when using the `env:` reference pattern.

## Common Pitfalls

1. **Storing API keys directly in context.json** — Passing `--api-key pk_live_abc123` directly stores the raw key in the config file. Always use the `env:` prefix to reference an environment variable instead.
2. **Forgetting which profile is active** — After switching profiles, commands silently target a different instance. Run `context show` before destructive operations to confirm you are pointing at the right environment.

## Best Practices

1. **Name profiles after environments** — Use names like `local`, `staging`, and `production` so it is immediately clear which instance a profile targets.
2. **Use `env:` references for all secrets** — This keeps API keys out of config files and shell history, reducing the risk of accidental exposure.

## Summary

- Profiles save CLI connection details so you do not repeat flags on every command.
- `context set` configures, `context show` displays, `context list` lists, and `context use` switches profiles.
- Profiles are stored at `~/.paperclip/context.json`.
- Use `env:VARIABLE_NAME` to reference API keys securely.
- Name profiles after their environment for clarity.

## Code Examples

**Creating and switching between context profiles**

```bash
# Set up a staging profile
pnpm paperclipai context set \
  --api-base http://staging:3100 \
  --company-id comp_abc123

# Switch profiles
pnpm paperclipai context use production

# Verify active profile
pnpm paperclipai context show
```


## Resources

- [Paperclip Configuration](https://paperclip.ing/docs) — Profile and context management documentation

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*