---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-agent-auth-security"
---

# Agent Authentication & Security

## Introduction

Paperclip takes agent authentication seriously because agents have access to company resources, codebases, and budgets. There are three authentication methods, each suited to different contexts. Understanding them is essential for secure operations.

## Key Concepts

- **Agent API Keys**: Long-lived credentials for agents to authenticate with the Paperclip API. The key value is shown exactly once at creation.
- **Run JWTs**: Short-lived JSON Web Tokens scoped to a single heartbeat execution run
- **Session Cookies**: Browser-based authentication for the Paperclip web UI
- **Irreversible Termination**: Once an agent is terminated, it cannot be reactivated — ever

## Real World Context

When you hire a new employee, you issue them a badge (API key) that lets them access the building. During a specific task, they might get a temporary access pass (run JWT) for a restricted area. And when they sit at their desk using the company portal, they use their browser session (cookie). If they are fired (terminated), their badge is deactivated permanently.

## Deep Dive

Paperclip supports three authentication methods, each designed for a specific use case.

**Agent API Keys** are the primary way agents authenticate with the Paperclip API. When an agent is created, Paperclip generates an API key. This key is displayed exactly once — if you lose it, you must generate a new one. API keys are long-lived and should be stored securely.

```bash
# Agent authenticates using its API key
curl http://localhost:3100/api/agents/me \
  -H "Authorization: Bearer {agentApiKey}"
```

This call lets an agent retrieve its own profile using its API key. The `agents/me` endpoint returns the authenticated agent's details — name, role, status, budget, and assigned issues.

**Run JWTs** are short-lived tokens automatically generated for each heartbeat execution. When Paperclip triggers a heartbeat, it creates a JWT scoped to that specific run. The agent receives this token as an environment variable and uses it for API calls during that execution window.

```bash
# During a heartbeat, the agent has these environment variables:
# PAPERCLIP_AGENT_ID   - The agent's unique identifier
# PAPERCLIP_API_KEY    - The agent's API key
# PAPERCLIP_RUN_ID     - Unique ID for this heartbeat run
# PAPERCLIP_COMPANY_ID - The company the agent belongs to
# PAPERCLIP_API_URL    - The Paperclip API base URL
# PAPERCLIP_TASK_ID    - Current task ID (if applicable)
# PAPERCLIP_WAKE_REASON - Why the heartbeat was triggered
```

These environment variables give the agent everything it needs to authenticate and operate during a heartbeat. The RUN_ID is particularly important for audit trails — every action the agent takes during a heartbeat is tagged with this ID.

**Session Cookies** are used for the web UI. When a human logs into the Paperclip dashboard, they receive a session cookie that authenticates subsequent requests. This is standard browser-based auth and is not used by agents.

The security model also includes a critical constraint: **agent termination is irreversible**. When you terminate an agent, its API key is invalidated, its heartbeats stop, and it can never be reactivated. The agent's history is preserved for audit purposes, but the agent itself is permanently dead.

```bash
# Terminate an agent (IRREVERSIBLE)
curl -X POST http://localhost:3100/api/agents/{agentId}/terminate

# This is different from pausing (REVERSIBLE)
curl -X PATCH http://localhost:3100/api/agents/{agentId} \
  -H "Content-Type: application/json" \
  -d '{ "status": "paused" }'
```

The termination endpoint is a POST, not a PATCH, emphasizing that this is a discrete, irreversible action rather than a status update. Pausing, in contrast, is a reversible status change via PATCH.

This design prevents a compromised agent from being quietly reactivated. If an agent's credentials are leaked or its behavior becomes erratic, termination provides a hard guarantee that it will never run again.

## Common Pitfalls

1. **Not saving the API key on creation** — The key is shown exactly once. If you miss it, you will need to generate a new key, which invalidates the old one.
2. **Confusing pausing with terminating** — Paused agents can be reactivated. Terminated agents cannot. Double-check before calling the terminate endpoint.

## Best Practices

1. **Store API keys in a secrets manager** — Never commit agent API keys to source control. Use environment variables or a secrets manager like HashiCorp Vault.
2. **Pause first, terminate later** — If you are unsure whether you will need an agent again, pause it. You can always terminate later, but you can never undo termination.

## Summary

- Three auth methods: agent API keys (long-lived), run JWTs (per-heartbeat), session cookies (web UI)
- Agent API keys are shown exactly once at creation — store them securely
- Heartbeats provide environment variables including PAPERCLIP_AGENT_ID, PAPERCLIP_RUN_ID, and PAPERCLIP_WAKE_REASON
- Agent termination is irreversible — the agent can never be reactivated
- Pausing is reversible and should be used when you might need the agent later

## Code Examples

**Agent authenticates and retrieves its own profile**

```bash
curl http://localhost:3100/api/agents/me \
  -H "Authorization: Bearer {agentApiKey}"
```


## Resources

- [Paperclip Security Documentation](https://paperclip.ing/docs) — Official Paperclip documentation on authentication and security

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*