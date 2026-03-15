---
source_course: "paperclip-api"
source_lesson: "paperclip-api-building-automations"
---

# Building Automations

## Introduction

The real power of the Paperclip API emerges when you combine multiple endpoints into automated workflows. This lesson shows how to build practical automations: auto-assigning issues, monitoring budgets, using CLI JSON output for scripts, and setting up webhook notifications.

## Key Concepts

- **Multi-Call Workflows**: Combining multiple API calls in sequence to automate complex processes.
- **Auto-Assignment**: Matching issues to agents based on capabilities and current workload.
- **Budget Monitoring**: Polling cost summaries and triggering alerts when thresholds are approached.
- **CLI JSON Output**: The `--json` flag on CLI commands outputs structured data for scripting.
- **Webhook Integration**: Using the HTTP adapter to send notifications to external services.

## Real World Context

You run a nightly automation that scans the backlog for unassigned issues, matches them to available agents, assigns them, and sends a Slack notification. If any agent is above 70% budget utilization, the automation flags them for review.

## Deep Dive

### Auto-Assign Issues

A workflow that matches unassigned backlog issues to agents:

```typescript
async function autoAssignIssues(companyId: string, token: string) {
  const headers = {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json',
  };

  // 1. Get unassigned backlog issues
  const issuesRes = await fetch(
    `http://localhost:3100/api/companies/${companyId}/issues?status=backlog`,
    { headers }
  );
  const issues = await issuesRes.json();
  const unassigned = issues.filter((i: any) => !i.assigneeId);

  // 2. Get active agents
  const agentsRes = await fetch(
    `http://localhost:3100/api/companies/${companyId}/agents`,
    { headers }
  );
  const agents = await agentsRes.json();
  const active = agents.filter((a: any) => a.status === 'active');

  // 3. Round-robin assignment
  for (let i = 0; i < unassigned.length; i++) {
    const agent = active[i % active.length];
    await fetch(`http://localhost:3100/api/issues/${unassigned[i].id}`, {
      method: 'PATCH', headers,
      body: JSON.stringify({ assigneeId: agent.id }),
    });
  }
  return { assigned: unassigned.length };
}
```

This function fetches unassigned issues and active agents, then assigns in a round-robin fashion.

### Budget Monitoring

```typescript
async function checkBudgets(companyId: string, token: string) {
  const res = await fetch(
    `http://localhost:3100/api/companies/${companyId}/costs/summary`,
    { headers: { 'Authorization': `Bearer ${token}`, 'Content-Type': 'application/json' } }
  );
  const summary = await res.json();
  if (summary.utilizationPercent >= 80) {
    console.warn(`Budget at ${summary.utilizationPercent}% utilization`);
  }
  return summary;
}
```

### CLI JSON Output

```bash
paperclip companies list --json | jq '.[] | .name'
paperclip agents list --company c1 --json | jq '.[] | select(.status == "active") | .name'
```

The `--json` flag outputs structured data that can be piped to jq.

### Webhook Notifications

```typescript
async function notifySlack(message: string, webhookUrl: string) {
  await fetch(webhookUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ text: message }),
  });
}
```

## Common Pitfalls

- **Not handling API errors in automation loops**: If one call fails, log the error and continue.
- **Polling too frequently**: Check cost summaries once per hour, not per minute.
- **Hardcoding agent IDs**: Use the agents list endpoint to discover agents dynamically.

## Best Practices

- **Build idempotent automations** that produce the same result when run multiple times.
- **Add logging to every automation step** for traceability.
- **Start with simple round-robin** and add capability matching as you scale.

## Summary

- Combine multiple API calls to build automated workflows
- Auto-assign issues by matching backlog items to active agents
- Monitor budget utilization and trigger alerts at threshold percentages
- Use --json CLI output for shell-based scripting with jq
- Integrate external notifications via webhooks
- Build idempotent, error-tolerant automations with proper logging

## Code Examples

**Budget monitoring function that warns when the 80% soft alert threshold is reached**

```typescript
async function checkBudgets(companyId: string, token: string) {
  const res = await fetch(
    `http://localhost:3100/api/companies/${companyId}/costs/summary`,
    { headers: { 'Authorization': `Bearer ${token}`, 'Content-Type': 'application/json' } }
  );
  const summary = await res.json();
  if (summary.utilizationPercent >= 80) {
    console.warn(`Budget at ${summary.utilizationPercent}%`);
  }
  return summary;
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering API automation patterns
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code with automation examples

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*