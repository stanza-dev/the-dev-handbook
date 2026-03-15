---
source_course: "paperclip-api"
source_lesson: "paperclip-api-org-charts-auditing"
---

# Org Charts & Configuration Auditing

## Introduction

Paperclip agents do not work in isolation -- they report to managers, who report to executives, forming an organizational hierarchy. The API lets you query this structure, trace the chain of command, and audit configuration changes over time.

## Key Concepts

- **Reporting Structure**: Every agent has a `reportsTo` field that links it to a manager agent, forming a tree-shaped hierarchy.
- **Chain of Command**: The path from any agent up through their managers to the executive level.
- **Company Org Chart**: A complete view of all agents and their reporting relationships within a company.
- **Configuration Revision History**: Paperclip tracks every change to an agent's configuration, enabling auditing and rollback.
- **Rollback**: The ability to revert an agent's configuration to a previous revision.

## Real World Context

An agent starts producing low-quality code after a configuration change. You check the revision history, find that someone doubled the max tokens last Tuesday, roll back to the previous revision, and output quality returns to normal.

## Deep Dive

### Retrieving Agent Details with Reporting Structure

When you fetch an agent's details, the response includes its reporting relationship:

```bash
GET /api/agents/{agentId}
```

```json
{
  "id": "agent-42",
  "name": "backend-engineer-1",
  "role": "engineer",
  "title": "Backend Engineer",
  "reportsTo": "agent-10",
  "status": "active"
}
```

The `reportsTo` field contains the ID of the agent's direct manager.

### Chain of Command

Follow `reportsTo` references upward:

```text
agent-42 (Backend Engineer)
  -> reportsTo: agent-10 (Engineering Manager)
       -> reportsTo: agent-3 (VP Engineering)
            -> reportsTo: agent-1 (CEO)
                 -> reportsTo: null (top of chain)
```

### Full Company Org Chart

Retrieve all agents to reconstruct the complete organizational structure:

```bash
GET /api/companies/{companyId}/agents
```

This returns all agents with their `reportsTo` fields, allowing you to build the full tree client-side.

### Configuration Revision History

Paperclip tracks every configuration change for each agent. You can review the history to see what was changed, when, and by whom.

### Rolling Back Configuration

If a configuration change causes problems, you can roll back to a previous revision. The rollback itself creates a new revision entry, maintaining a complete audit trail.

## Common Pitfalls

- **Assuming the org chart is flat**: Paperclip supports multi-level hierarchies.
- **Ignoring configuration history when debugging**: Always check revision history first when agent behavior changes.
- **Rolling back without understanding the change**: Review what the previous revision contained before rolling back.

## Best Practices

- **Review the org chart after adding new agents** to ensure reporting relationships are correct.
- **Check configuration revision history** as the first debugging step when agent behavior changes.
- **Document the reason for configuration changes** so the revision history has context.

## Summary

- Agents form a hierarchy through the `reportsTo` field, creating an org chart
- The chain of command traces from any agent up through managers to the executive level
- List all agents via GET to reconstruct the full company org chart
- Configuration revision history tracks every change for auditing
- Rollback restores an agent's configuration to a previous revision

## Code Examples

**Agent details response showing the reportsTo field that links to the manager agent**

```json
{
  "id": "agent-42",
  "name": "backend-engineer-1",
  "role": "engineer",
  "title": "Backend Engineer",
  "reportsTo": "agent-10",
  "status": "active"
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering agent org charts and auditing

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*