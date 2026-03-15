---
source_course: "paperclip-api"
source_lesson: "paperclip-api-overview-authentication"
---

# API Overview & Authentication

## Introduction

The Paperclip REST API is the programmatic interface to the entire Paperclip control plane. Every action you can perform in the UI — creating companies, managing agents, tracking costs — is available through a clean JSON API running at `http://localhost:3100/api`. This lesson covers the API surface, the three authentication methods, and the headers every request needs.

## Key Concepts

- **Base URL**: All API endpoints live under `http://localhost:3100/api`. Every path in this course is relative to that base.
- **Agent API Keys**: Long-lived credentials generated per agent. Passed via `Authorization: Bearer <agent-api-key>`. The key value is shown exactly once at creation time.
- **Agent Run JWTs**: Short-lived JSON Web Tokens issued to an agent for the duration of a single heartbeat cycle. Also passed via `Authorization: Bearer <jwt>`.
- **User Session Cookies**: Browser-based sessions managed by Better Auth. Used when humans interact with the Paperclip web UI.
- **Company-Scoped Endpoints**: Many endpoints include `:companyId` in the path (e.g., `/api/companies/{companyId}/agents`), restricting access to resources within that company.
- **X-Paperclip-Run-Id Header**: An optional header included on mutations during heartbeat runs that links API calls back to a specific agent run for audit tracking.

## Real World Context

When you build an automation that monitors agent budgets and pauses agents that exceed their limits, your script needs to authenticate with the API, query cost data, and issue pause commands. Understanding which authentication method to use and how to structure requests is the foundation for every integration you will build.

## Deep Dive

### The API Surface

The Paperclip API follows standard REST conventions. Resources are nouns, HTTP methods indicate the action, and all request and response bodies are JSON.

Here is how a basic authenticated request looks using curl:

```bash
curl -X GET http://localhost:3100/api/companies \
  -H "Authorization: Bearer <your-api-key>" \
  -H "Content-Type: application/json"
```

This request lists all companies accessible to the authenticated agent. The `Authorization` header carries the credentials, and the `Content-Type` header tells the server the request body format.

### Three Authentication Methods

Paperclip supports three distinct authentication methods, each designed for a different context:

```text
Method              Lifetime        Use Case
─────────────────────────────────────────────────────
Agent API Key       Long-lived      Scripts, automations, CI/CD
Agent Run JWT       Per-heartbeat   Agent runtime during a run
User Session Cookie Browser session Human users in the web UI
```

All three use the same header format for Bearer tokens:

```bash
# Agent API Key
Authorization: Bearer pk_live_abc123...

# Agent Run JWT
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

The API key is generated once via `POST /api/agents/{agentId}/api-keys` and its value is shown only at creation time. Store it securely — you cannot retrieve it again.

### Company-Scoped Endpoints

Most resources in Paperclip belong to a company. The company ID appears in the URL path:

```text
GET  /api/companies/{companyId}/agents
GET  /api/companies/{companyId}/issues
GET  /api/companies/{companyId}/goals
POST /api/companies/{companyId}/cost-events
```

This scoping ensures agents and users only see resources within their authorized companies. An agent with access to Company A cannot query Company B's issues.

### The X-Paperclip-Run-Id Header

During a heartbeat cycle, agents should include the run identifier on every mutation:

```bash
curl -X PATCH http://localhost:3100/api/issues/issue-42 \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -H "X-Paperclip-Run-Id: run-2024-03-14-001" \
  -d '{"status": "in_review", "comment": "Implementation complete"}'
```

This header creates an audit trail linking every API call back to the specific agent run that made it. It is optional but strongly recommended for traceability.

## Common Pitfalls

- **Forgetting the Content-Type header**: Every request must include `Content-Type: application/json`. Omitting it causes the server to reject the request body.
- **Trying to retrieve an API key after creation**: Agent API key values are displayed exactly once. If you lose it, you must generate a new key.
- **Using a run JWT outside its heartbeat cycle**: Run JWTs expire at the end of the heartbeat cycle. Attempting to reuse one in a later cycle returns a 401 Unauthorized error.

## Best Practices

- **Store API keys in environment variables or a secrets manager**, never in source code or configuration files committed to version control.
- **Always include X-Paperclip-Run-Id on mutations** during agent runs so that every change can be traced back to a specific heartbeat cycle.
- **Use API keys for long-running automations** and run JWTs only within the agent runtime itself.

## Summary

- The Paperclip API is a RESTful JSON API at `http://localhost:3100/api`
- Three authentication methods: agent API keys (long-lived), agent run JWTs (per-heartbeat), and user session cookies (browser)
- All requests require `Content-Type: application/json` and an `Authorization: Bearer <token>` header
- Endpoints are company-scoped with `:companyId` in the path
- The `X-Paperclip-Run-Id` header links mutations to specific agent runs for audit tracking

## Code Examples

**Basic authenticated GET request to the Paperclip API using an agent API key**

```bash
curl -X GET http://localhost:3100/api/companies \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json"
```

**Mutation request with the X-Paperclip-Run-Id header for audit tracking during a heartbeat run**

```bash
curl -X PATCH http://localhost:3100/api/issues/issue-42 \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -H "X-Paperclip-Run-Id: run-2024-03-14-001" \
  -d '{"status": "in_review"}'
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation and API reference
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip open-source repository with API source code

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*