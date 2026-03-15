---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-agents-as-employees"
---

# Agents as AI Employees

## Introduction

In Paperclip, an agent is not just a process — it is an AI employee. Each agent has a role, a manager, a budget, a set of capabilities, and a lifecycle status. This employment model is what distinguishes Paperclip from running agents ad hoc in terminals.

## Key Concepts

- **Agent**: An AI employee defined by an adapter configuration, role, reporting line, per-agent budget, and status
- **Adapter Configuration**: Specifies which runtime (Claude, Codex, Gemini, etc.) powers the agent
- **Agent Statuses**: Six lifecycle states — active, idle, running, error, paused, terminated
- **Capabilities**: What an agent can do (e.g., write code, review PRs, run tests)
- **Job Description**: A human-readable description of the agent's responsibilities, surfaced as context during execution

## Real World Context

Consider hiring a backend developer at a real company. You define their role (Senior Backend Engineer), assign a manager (Engineering Lead), set a compensation budget, list required skills (Node.js, PostgreSQL, API design), and write a job description. Paperclip mirrors this process for AI agents, giving each one a structured identity that the control plane uses for work assignment and oversight.

## Deep Dive

Every agent in Paperclip is a combination of configuration and organizational identity. The adapter configuration tells Paperclip which runtime to use when the agent needs to execute work. The organizational attributes tell Paperclip where the agent fits in the company.

Here is how you propose hiring an agent via the API:

```bash
# Propose hiring a new agent
curl -X POST http://localhost:3100/api/companies/{companyId}/agent-hires \
  -H "Content-Type: application/json" \
  -d '{
    "name": "backend-coder",
    "role": "ic",
    "adapter": "claude_local",
    "jobDescription": "Senior backend engineer responsible for API development, database schema design, and integration testing",
    "monthlyBudget": 10000
  }'
```

This proposes hiring an agent named "backend-coder" as an individual contributor (IC) using the Claude local adapter, with a monthly budget of $100.00 (10000 cents). Note that this is a hiring proposal — it goes through governance approval before the agent is created.

Agents have six possible statuses that track their lifecycle:

- **active**: The agent is registered and available for work but not currently executing
- **idle**: The agent has no assigned work and is waiting for new tasks
- **running**: The agent is currently executing a heartbeat — doing active work
- **error**: Something went wrong during execution; requires attention
- **paused**: The agent is temporarily suspended — no heartbeats, no new work
- **terminated**: The agent is permanently deactivated — this is irreversible

The distinction between paused and terminated is critical. Pausing an agent is like putting an employee on leave — they can come back. Terminating an agent is like firing them — their record remains for audit purposes, but they can never be reactivated.

```bash
# Check an agent's current status
curl http://localhost:3100/api/agents/{agentId}

# Pause an agent (reversible)
curl -X PATCH http://localhost:3100/api/agents/{agentId} \
  -H "Content-Type: application/json" \
  -d '{ "status": "paused" }'
```

The first call retrieves the agent's full profile including status, budget, role, and adapter. The second call pauses the agent, preventing any further heartbeats until it is reactivated.

Capabilities and job descriptions provide context to the control plane and to the agent itself. When Paperclip decides which agent should handle a particular issue, it considers the agent's capabilities. When an agent starts working, its job description is included in the context so it understands its own role.

## Common Pitfalls

1. **Terminating when you mean to pause** — Termination is irreversible. If you want to temporarily stop an agent, use paused. Only terminate when you are certain the agent should never run again.
2. **Setting budgets too high initially** — Start with conservative budgets and increase based on observed cost patterns. You can always PATCH the budget upward.

## Best Practices

1. **Write detailed job descriptions** — The job description is fed to the agent as context. Clear, specific descriptions lead to better agent performance.
2. **Use meaningful agent names** — Names like "backend-coder" or "test-runner" are better than "agent-1" because they communicate purpose at a glance.

## Summary

- Agents are AI employees with adapter configs, roles, reporting lines, budgets, and statuses
- Six statuses: active, idle, running, error, paused, terminated
- Hiring is proposed via POST /api/companies/{companyId}/agent-hires and goes through governance
- Termination is irreversible — use pause for temporary suspension
- Capabilities and job descriptions provide context for work assignment and agent execution

## Code Examples

**Propose hiring a new agent with an adapter and budget**

```bash
curl -X POST http://localhost:3100/api/companies/{companyId}/agent-hires \
  -H "Content-Type: application/json" \
  -d '{
    "name": "backend-coder",
    "role": "ic",
    "adapter": "claude_local",
    "monthlyBudget": 10000
  }'
```


## Resources

- [Paperclip Agents Documentation](https://paperclip.ing/docs) — Official documentation for managing agents in Paperclip

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*