---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-control-plane"
---

# Control Plane Commands

## Introduction

The control-plane commands let you manage every aspect of a Paperclip company from the terminal. You can create and track issues, inspect agents, approve or reject pending actions, and export entire company configurations. Combined with the `--json` flag, these commands enable powerful scripting and automation.

## Key Concepts

- **issue**: Manage work items — `list`, `get`, `create`, `update`, `comment`, `checkout`, `release`.
- **agent**: Inspect agents — `list`, `get`.
- **approval**: Handle pending agent actions — `list`, `approve`, `reject`.
- **company**: Manage companies — `list`, `get`, `export`, `import`.
- **--json**: Machine-readable output for scripting.

## Real World Context

A team lead starts their morning by reviewing what agents accomplished overnight. Instead of opening the web UI, they run `pnpm paperclipai issue list --status done --json | jq '.[] | .title'` to see completed issues, then `pnpm paperclipai approval list` to approve or reject any pending agent actions. This terminal-first workflow integrates naturally with existing developer habits and tooling.

## Deep Dive

The `issue` command is the most frequently used control-plane command. It mirrors a typical issue tracker.

```bash
# List all issues
pnpm paperclipai issue list

# Get details of a specific issue
pnpm paperclipai issue get --id issue_abc123

# Create a new issue
pnpm paperclipai issue create \
  --title "Add user authentication" \
  --description "Implement login and signup flows"

# Update an issue
pnpm paperclipai issue update --id issue_abc123 --status in_progress

# Add a comment
pnpm paperclipai issue comment --id issue_abc123 \
  --body "Blocked on API key provisioning"

# Checkout an issue (assign to self/agent)
pnpm paperclipai issue checkout --id issue_abc123

# Release an issue (unassign)
pnpm paperclipai issue release --id issue_abc123
```

The `agent` command lets you inspect the agents in your company.

```bash
# List all agents
pnpm paperclipai agent list

# Get details of a specific agent
pnpm paperclipai agent get --id agent_xyz789
```

The `approval` command handles the human-in-the-loop workflow. When an agent needs permission to perform a sensitive action, it creates an approval request.

```bash
# List pending approvals
pnpm paperclipai approval list

# Approve an action
pnpm paperclipai approval approve --id appr_def456

# Reject an action
pnpm paperclipai approval reject --id appr_def456 \
  --reason "Budget exceeded for this sprint"
```

The `company` command manages organizational configuration.

```bash
# List all companies
pnpm paperclipai company list

# Export company configuration
pnpm paperclipai company export --id comp_abc123 > backup.json

# Import company configuration
pnpm paperclipai company import --file backup.json
```

Combine `--json` with shell tools for powerful automation.

```bash
#!/bin/bash
# Auto-approve all low-risk approvals
pnpm paperclipai --json approval list | \
  jq -r '.[] | select(.risk == "low") | .id' | \
  while read id; do
    pnpm paperclipai approval approve --id "$id"
  done
```

This script lists all pending approvals as JSON, filters for low-risk items using `jq`, and approves each one automatically.

## Common Pitfalls

1. **Auto-approving without filtering by risk** — Approving all pending actions blindly can let agents perform destructive operations. Always filter by risk level or review individually for high-risk actions.
2. **Forgetting to set --company-id** — Control-plane commands need a company scope. Without it, you get empty results or errors. Use context profiles to set a default.

## Best Practices

1. **Use `company export` for backups** — Before making major changes, export the company configuration. This creates a complete snapshot that can be imported to restore or replicate the setup.
2. **Pipe --json output to jq for filtering** — The JSON output contains all fields. Use `jq` to extract exactly what you need, making scripts concise and maintainable.

## Summary

- `issue` commands manage work items: create, update, comment, checkout, and release.
- `agent` commands inspect agent configuration and status.
- `approval` commands handle human-in-the-loop decisions.
- `company` commands manage organizational config with export/import for backups.
- Combine `--json` with shell tools like `jq` for automation scripts.

## Code Examples

**Common control-plane operations for managing issues and approvals**

```bash
# Create an issue and list pending approvals
pnpm paperclipai issue create --title "Fix auth bug"
pnpm paperclipai approval list

# Export company config as backup
pnpm paperclipai company export --id comp_abc123 > backup.json
```


## Resources

- [Paperclip CLI Commands](https://paperclip.ing/docs) — Full reference for all CLI commands and subcommands

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*