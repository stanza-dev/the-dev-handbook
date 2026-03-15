---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-cost-control-budgets"
---

# Cost Control & Budgets

## Introduction

AI agents consume API credits every time they run. Without cost controls, a single runaway agent can burn through your entire budget overnight. Paperclip provides granular budget management at the agent and company level, with automatic alerts and hard stops to prevent overspending.

## Key Concepts

- **Per-Agent Budget**: A monthly spending limit in cents assigned to each individual agent
- **Company-Wide Budget**: An aggregate monthly limit across all agents in a company
- **Cost Events**: Individual spend records tracking provider, model, tokens, and cost
- **80% Soft Alert**: A warning notification when an agent reaches 80% of its monthly budget
- **100% Hard Stop**: Automatic agent pausing when the full budget is exhausted
- **Monthly Reset**: Budget counters reset on the 1st of each month, UTC

## Real World Context

Think of budgets like a company's departmental spending limits. Each department (agent) has its own budget. The CFO (Paperclip) monitors spending in real-time. At 80% of budget, the department gets a warning. At 100%, the department's spending is frozen until next month. The company also has an overall budget that caps aggregate spending across all departments.

## Deep Dive

Budgets in Paperclip are specified in **cents**, not dollars. This avoids floating-point precision issues that can cause accounting errors. A budget of 10000 means $100.00.

Every agent has a per-agent monthly budget set at creation:

```bash
# Agent with $100/month budget
# monthlyBudget: 10000 (cents)

# Company with $500/month budget
# monthlyBudget: 50000 (cents)
```

As agents execute heartbeats, they consume API credits. Each consumption is recorded as a **cost event** that captures the provider (e.g., Anthropic), model (e.g., Claude Sonnet), token counts (input and output), and the dollar cost.

Paperclip provides cost reporting at three levels:

```bash
# 1. Company summary — total spend across all agents
curl http://localhost:3100/api/companies/{companyId}/costs

# 2. Agent breakdown — spend per individual agent
curl http://localhost:3100/api/companies/{companyId}/costs/agents

# 3. Project breakdown — spend per project
curl http://localhost:3100/api/companies/{companyId}/costs/projects
```

The company summary shows the aggregate spend. The agent breakdown shows which agents are consuming the most budget. The project breakdown shows which projects are the most expensive. Together, these three views give you complete visibility into where money is going.

The budget enforcement system has two thresholds:

**80% Soft Alert**: When an agent's spending reaches 80% of its monthly budget, Paperclip generates a warning. This alert surfaces in the UI and can trigger notifications. It is a heads-up that the agent is approaching its limit.

**100% Hard Stop**: When an agent hits 100% of its monthly budget, Paperclip automatically pauses the agent. The agent's status moves to paused, and no more heartbeats are triggered until the budget resets. This prevents any overspending.

```bash
# Budget enforcement example:
# Agent monthly budget: $100 (10000 cents)
# At $80 spent  → 80% soft alert (warning)
# At $100 spent → 100% hard stop (agent auto-paused)
# On the 1st of next month (UTC) → budget resets, agent can resume
```

Budget cycles reset on the **1st of each month at midnight UTC**. When the reset happens, spending counters return to zero and paused agents whose only pause reason was budget exhaustion can resume operations.

The company-wide budget provides an additional safety net. Even if individual agents have remaining budget, if the company's aggregate spending hits its limit, all agents in that company are affected.

## Common Pitfalls

1. **Confusing cents with dollars** — A budget of 5000 is $50.00, not $5,000. Always divide by 100 to convert cents to dollars.
2. **Forgetting about the monthly reset date** — Budgets reset on the 1st UTC, not your local timezone. An agent paused on the 31st may resume before your local midnight.

## Best Practices

1. **Set conservative initial budgets** — Start low and increase based on observed usage. It is easier to raise a budget than to recover from overspending.
2. **Monitor the 80% alerts** — Treat the soft alert as a trigger to review the agent's work efficiency. High spending might indicate inefficient prompts or unnecessary retries.

## Summary

- Budgets are in cents to avoid floating-point issues (10000 = $100.00)
- Per-agent and company-wide monthly budgets provide granular and aggregate control
- Cost events track provider, model, tokens, and dollar cost per execution
- Three reporting levels: company summary, agent breakdown, project breakdown
- 80% soft alert warns of approaching limits; 100% hard stop auto-pauses the agent
- Budget cycles reset on the 1st of each month at midnight UTC

## Code Examples

**Query cost reports at company and agent levels**

```bash
# Check company-wide costs
curl http://localhost:3100/api/companies/{companyId}/costs

# Check per-agent cost breakdown
curl http://localhost:3100/api/companies/{companyId}/costs/agents
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation on budgets and cost management

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*