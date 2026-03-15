---
source_course: "paperclip-api"
source_lesson: "paperclip-api-cost-tracking"
---

# Cost Tracking API

## Introduction

Every AI model call costs money, and Paperclip tracks every cent. The Cost Tracking API lets you log cost events, query spending summaries, and break down costs by agent or project. Combined with budget enforcement, it ensures no agent exceeds its financial limits.

## Key Concepts

- **Cost Events**: Individual records of AI model usage, including the agent, provider, model, token counts, and cost in cents.
- **Company Cost Summary**: Aggregated spending data including total spent, budget, and utilization percentage.
- **Agent Breakdown**: Per-agent cost allocation showing how much each agent has spent.
- **Project Breakdown**: Cost distribution by project for project-level budget tracking.
- **Progressive Enforcement**: At 80% budget utilization, agents receive a soft alert. At 100%, agents are hard-stopped (auto-paused).

## Real World Context

You have a company with a $1,000/month budget and ten agents. Halfway through the month, utilization is at 65%. The cost breakdown reveals that one agent is responsible for 40% of total spending. You reduce that agent's budget and switch it to a more cost-effective model.

## Deep Dive

### Logging Cost Events

```bash
POST /api/companies/{companyId}/cost-events
```

```json
{
  "agentId": "agent-42",
  "provider": "anthropic",
  "model": "claude-sonnet-4-20250514",
  "inputTokens": 15000,
  "outputTokens": 3000,
  "costCents": 12
}
```

Each event records which agent made the call, the provider and model used, token counts, and cost in cents.

### Company Cost Summary

```bash
GET /api/companies/{companyId}/costs/summary
```

```json
{
  "totalSpentCents": 45000,
  "budgetMonthlyCents": 100000,
  "utilizationPercent": 45,
  "periodStart": "2024-03-01T00:00:00Z",
  "periodEnd": "2024-03-31T23:59:59Z"
}
```

The `utilizationPercent` shows what percentage of the monthly budget has been consumed. The period resets on the 1st of each month (UTC).

### Cost Breakdown by Agent and Project

```bash
GET /api/companies/{companyId}/costs/by-agent
GET /api/companies/{companyId}/costs/by-project
```

These endpoints let you identify which agents are the highest spenders and how costs distribute across projects.

### Budget Management

Set company and agent budgets via PATCH:

```bash
PATCH /api/companies/{companyId}
{"budgetMonthlyCents": 200000}

PATCH /api/agents/{agentId}
{"budgetMonthlyCents": 10000}
```

Budgets are in cents. Setting 200000 means $2,000.00 per month.

### Progressive Enforcement

Paperclip enforces budgets progressively:

```text
  0% ----------- 80% ------------ 100%
  |               |                 |
  |  Normal ops   |  Soft alert:    |  Hard stop:
  |               |  Focus on       |  Agent auto-paused
  |               |  critical tasks |
```

At 80% utilization, the agent receives a soft alert instructing it to focus on critical tasks only. At 100% utilization, the agent is automatically paused.

## Common Pitfalls

- **Logging costs in dollars instead of cents**: The `costCents` field is in cents. A cost of 12 means $0.12, not $12.00.
- **Ignoring the 80% soft alert**: It is a warning that spending is on pace to exceed the budget.
- **Not setting individual agent budgets**: Without agent-level budgets, a single agent can consume the entire company budget.

## Best Practices

- **Log cost events for every model call** to maintain accurate spending data.
- **Monitor utilization daily** and adjust budgets proactively.
- **Set agent budgets proportionally** based on expected workload and model costs.

## Summary

- Cost events record agent, provider, model, tokens, and cost in cents
- Company cost summary shows total spending, budget, and utilization percentage
- Cost breakdowns are available by agent and by project
- Budgets are managed via PATCH on companies and agents (in cents)
- Progressive enforcement: 80% soft alert, 100% hard stop (auto-pause)
- Monthly period resets on the 1st of each month (UTC)

## Code Examples

**Cost event payload recording an AI model call with provider, model, token counts, and cost in cents**

```json
{
  "agentId": "agent-42",
  "provider": "anthropic",
  "model": "claude-sonnet-4-20250514",
  "inputTokens": 15000,
  "outputTokens": 3000,
  "costCents": 12
}
```

**Company cost summary response showing 45% budget utilization for the current month**

```json
{
  "totalSpentCents": 45000,
  "budgetMonthlyCents": 100000,
  "utilizationPercent": 45,
  "periodStart": "2024-03-01T00:00:00Z",
  "periodEnd": "2024-03-31T23:59:59Z"
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering cost tracking and budget management
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code with cost tracking implementation

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*