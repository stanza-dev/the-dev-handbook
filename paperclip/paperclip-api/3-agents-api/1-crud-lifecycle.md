---
source_course: "paperclip-api"
source_lesson: "paperclip-api-agent-crud-lifecycle"
---

# Agent CRUD & Lifecycle

## Introduction

Agents are the autonomous workers in Paperclip. They check out issues, write code, log costs, and report status through the API. Managing their lifecycle programmatically is essential for building control plane automations. This lesson covers creating agents, configuring them, controlling their state, generating API keys, and triggering heartbeats.

## Key Concepts

- **Agent Creation**: Agents are created within a company with a name, role, title, capabilities, adapter configuration, and budget.
- **Adapter Config**: Specifies which AI model provider the agent uses (e.g., `claude_local`, `codex_local`) and its configuration.
- **Lifecycle Control**: Agents can be paused, resumed, and terminated via dedicated endpoints.
- **API Key Generation**: Each agent can have API keys generated for programmatic access. The key value is shown only once.
- **Manual Heartbeat**: You can trigger an agent's heartbeat cycle manually via the API.

## Real World Context

You are scaling up your engineering team of AI agents. You need to create three new agents, each with different roles and model configurations. After creation, you generate API keys for each so they can authenticate independently. When an agent misbehaves, you pause it immediately via the API, investigate, and either resume or terminate it.

## Deep Dive

### Listing Agents

Retrieve all agents in a company:

```bash
GET /api/companies/{companyId}/agents
```

The response is an array of agent objects with their IDs, names, roles, status, and configuration.

### Creating an Agent

Create a new agent with its full configuration:

```bash
POST /api/companies/{companyId}/agents
```

```json
{
  "name": "backend-engineer-1",
  "role": "engineer",
  "title": "Backend Engineer",
  "reportsTo": "manager-agent-id",
  "capabilities": "TypeScript, Node.js, PostgreSQL, API development",
  "adapterType": "claude_local",
  "adapterConfig": {
    "model": "claude-sonnet-4-20250514",
    "maxTokens": 8192
  },
  "budgetMonthlyCents": 5000
}
```

The `reportsTo` field establishes the agent's position in the org chart. The `adapterType` and `adapterConfig` specify which AI model the agent uses. The `budgetMonthlyCents` sets the agent's individual spending limit.

### Updating an Agent

Update an agent's configuration:

```bash
PATCH /api/agents/{agentId}
```

```json
{
  "budgetMonthlyCents": 10000,
  "adapterConfig": {
    "model": "claude-sonnet-4-20250514",
    "maxTokens": 16384
  }
}
```

PATCH is idempotent -- sending the same update twice produces the same result.

### Lifecycle Control

Three endpoints control an agent's lifecycle:

```bash
POST /api/agents/{agentId}/pause      # Temporarily stop
POST /api/agents/{agentId}/resume     # Reactivate
POST /api/agents/{agentId}/terminate  # Permanent shutdown
```

Pausing is reversible. Termination is permanent.

### Generating API Keys

Create an API key for an agent:

```bash
POST /api/agents/{agentId}/api-keys
```

The response includes the key value, displayed exactly once:

```json
{
  "id": "key-1",
  "value": "pk_live_abc123def456...",
  "createdAt": "2024-03-14T10:00:00Z"
}
```

Store this value immediately. There is no way to retrieve it again.

### Manual Heartbeat

Trigger an agent's heartbeat cycle:

```bash
POST /api/agents/{agentId}/heartbeat
```

This starts the agent's work cycle: it checks for assigned issues, performs work, logs costs, and reports status.

## Common Pitfalls

- **Not storing the API key immediately**: The key value is shown once. If you lose it, you must generate a new key.
- **Using terminate when you mean pause**: Termination is permanent. Use pause for temporary shutdowns.
- **Forgetting to set the adapter configuration**: An agent without an adapter config cannot perform work.

## Best Practices

- **Store API keys in a secrets manager** immediately after generation.
- **Set conservative budgets initially** and increase them as you validate behavior.
- **Use pause for investigation** before deciding whether to resume or terminate.

## Summary

- Agents are created with name, role, capabilities, adapter config, and budget
- Lifecycle control: pause (reversible), resume, terminate (permanent)
- API keys are generated via POST and the value is shown exactly once
- Manual heartbeats trigger an agent's work cycle on demand
- PATCH updates are idempotent and can modify budget, adapter config, and role

## Code Examples

**Full agent creation payload with role, capabilities, adapter configuration, and budget**

```json
{
  "name": "backend-engineer-1",
  "role": "engineer",
  "title": "Backend Engineer",
  "reportsTo": "manager-agent-id",
  "capabilities": "TypeScript, Node.js, PostgreSQL",
  "adapterType": "claude_local",
  "adapterConfig": { "model": "claude-sonnet-4-20250514", "maxTokens": 8192 },
  "budgetMonthlyCents": 5000
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering the Agents API
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code with Agents API implementation

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*