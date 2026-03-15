---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-advanced-config"
---

# Advanced Agent Configuration

## Introduction

As your Paperclip deployment grows, managing agent configurations becomes critical. Paperclip provides configuration auditing, safe rollback, and portable templates to help teams manage configs at scale. These features ensure that configuration changes are traceable, reversible, and shareable.

## Key Concepts

- **Configuration Auditing**: Every change to an agent's configuration is recorded with a timestamp, author, and diff
- **Safe Rollback**: Revert to any previous configuration version without losing the current one
- **Versioned Configs**: Each configuration save creates a new version, building a complete revision history
- **Portable Templates**: Export company configurations (with secret scrubbing) for sharing or backup
- **ClipHub**: A planned marketplace for pre-built company templates (coming soon)

## Real World Context

A team deploys a new prompt template for their coding agent on Monday. By Wednesday, they notice the agent's code quality has degraded. Without configuration versioning, they would need to reconstruct the old prompt from memory or Git history. With Paperclip's revision history, they can view the exact diff, identify the problematic change, and roll back in seconds.

## Deep Dive

### Configuration Auditing

Every time you save an agent's configuration, Paperclip creates a new revision with metadata:

```json
{
  "revision": 5,
  "timestamp": "2026-03-14T10:30:00Z",
  "author": "dev@company.com",
  "diff": {
    "changed": {
      "adapterConfig.model": {
        "from": "claude-sonnet-4-20250514",
        "to": "claude-opus-4-20250514"
      },
      "adapterConfig.timeoutSec": {
        "from": 300,
        "to": 600
      }
    }
  }
}
```

The revision record shows exactly what changed, who changed it, and when. This audit trail is invaluable for debugging agent behavior changes and for compliance requirements.

### Safe Rollback

Rollback uses versioned configs for safe reversal. When you roll back, Paperclip does not delete the current version — it creates a new revision that copies the old configuration:

```json
{
  "revision": 6,
  "timestamp": "2026-03-14T14:00:00Z",
  "author": "dev@company.com",
  "rollbackFrom": 5,
  "rollbackTo": 3,
  "note": "Rolling back model change — code quality regression"
}
```

This approach means you can always roll forward again if needed. No configuration is ever permanently lost.

The rollback flow in the UI is straightforward: view the revision history, select the version you want, click rollback, and optionally add a note explaining why. The system creates a new revision with the old configuration's values.

### Portable Templates

Paperclip can export entire company configurations as portable templates. During export, all `secret_ref` values are scrubbed to prevent credential leakage:

```json
{
  "template": {
    "name": "Acme Corp Coding Agent",
    "version": "1.0.0",
    "agents": [
      {
        "name": "Code Reviewer",
        "adapterType": "claude_local",
        "adapterConfig": {
          "model": "claude-sonnet-4-20250514",
          "timeoutSec": 300,
          "env": {
            "ANTHROPIC_API_KEY": "<SECRET_SCRUBBED>"
          }
        }
      }
    ]
  }
}
```

Notice that the API key has been replaced with `<SECRET_SCRUBBED>`. When importing a template, Paperclip prompts the user to provide their own secrets. This makes templates safe to share across teams, companies, or publish publicly.

ClipHub is a planned marketplace where teams will be able to share and discover pre-built company templates. It is not yet available but is on the Paperclip roadmap.

## Common Pitfalls

- **Not adding rollback notes**: Without a note, it is hard to understand why a rollback happened when reviewing the history later.
- **Assuming rollback deletes the current config**: Rollback creates a new revision. The old configuration is preserved in the history.
- **Sharing exported templates without checking for secrets**: Always verify that `secret_ref` values are scrubbed before sharing templates externally.

## Best Practices

- **Review the diff before rolling back**: The diff view shows exactly what will change. Verify it matches your expectations before confirming.
- **Export templates before major changes**: Create a portable backup of your configuration before making significant modifications.
- **Use meaningful revision notes**: When saving configuration changes, add a note explaining the intent (e.g., "Switching to Opus for complex refactoring tasks").

## Summary

- Every configuration change creates a new revision with timestamp, author, and diff
- Rollback creates a new revision with the old configuration — nothing is deleted
- Versioned configs provide a complete audit trail for compliance and debugging
- Portable templates export company configs with secrets scrubbed for safe sharing
- ClipHub will provide a marketplace for pre-built templates (coming soon)

## Code Examples

**A rollback revision record showing that the system creates a new revision rather than deleting the current configuration**

```json
{
  "revision": 6,
  "timestamp": "2026-03-14T14:00:00Z",
  "author": "dev@company.com",
  "rollbackFrom": 5,
  "rollbackTo": 3,
  "note": "Rolling back model change — code quality regression"
}
```


## Resources

- [Paperclip Configuration Management](https://paperclip.ing/docs/configuration) — Official documentation for Paperclip configuration versioning, auditing, and rollback
- [Paperclip Templates Guide](https://paperclip.ing/docs/templates) — Guide to exporting, importing, and sharing portable company templates

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*