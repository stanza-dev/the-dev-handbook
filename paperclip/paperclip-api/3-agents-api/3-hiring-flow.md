---
source_course: "paperclip-api"
source_lesson: "paperclip-api-agent-hiring-flow"
---

# Agent Hiring Flow

## Introduction

Creating an agent directly is one path, but Paperclip also supports a formal hiring flow that mimics real-world HR processes. A hire request goes through board review, and only after approval is the agent fully activated.

## Key Concepts

- **Hire Request**: A proposal to add a new agent, submitted via `POST /api/companies/{companyId}/agent-hires`.
- **Draft Agent**: When a hire request is created, a draft agent is automatically created alongside an approval request.
- **Board Review**: The hire request is reviewed by the board who can approve, reject, or request revisions.
- **Post-Approval Configuration**: After approval, the new agent needs its adapter and secrets configured.

## Real World Context

Your engineering manager agent determines that the team needs a QA specialist. Instead of creating the agent directly, the manager submits a hire request. The CEO agent reviews the request, checks the budget impact, and approves it. The new QA agent is activated and configured.

## Deep Dive

### Submitting a Hire Request

```bash
POST /api/companies/{companyId}/agent-hires
```

```json
{
  "role": "engineer",
  "title": "QA Specialist",
  "reportsTo": "manager-agent-id",
  "capabilities": "Testing, code review, test automation, CI/CD",
  "budgetMonthlyCents": 5000
}
```

This creates two things simultaneously: a draft agent with the specified attributes, and an approval request for the board to review.

### The Approval Flow

After submission, the hire request enters the approval pipeline:

```text
1. Hire request submitted -> Draft agent created
2. Approval request created -> Board notified
3. Board reviews -> Approves, Rejects, or Requests Revision
4. If revision requested -> Resubmit with changes
5. If approved -> Agent activated
6. Post-approval -> Configure adapter and secrets
```

The board can take three actions:

```bash
POST /api/approvals/{approvalId}/approve
POST /api/approvals/{approvalId}/reject
POST /api/approvals/{approvalId}/request-revision
```

If revisions are requested, resubmit:

```bash
POST /api/approvals/{approvalId}/resubmit
```

### Post-Approval Configuration

Once approved, configure the adapter and generate an API key:

```bash
PATCH /api/agents/{newAgentId}
```

```json
{
  "adapterType": "claude_local",
  "adapterConfig": {
    "model": "claude-sonnet-4-20250514",
    "maxTokens": 8192
  }
}
```

Then generate an API key:

```bash
POST /api/agents/{newAgentId}/api-keys
```

## Common Pitfalls

- **Skipping the hiring flow for production agents**: In governed environments, creating agents directly bypasses review and budget controls.
- **Forgetting post-approval configuration**: An approved agent without adapter config and API keys cannot do work.
- **Not handling rejection gracefully**: Address feedback before resubmitting.

## Best Practices

- **Include clear justification** in the hire request capabilities field.
- **Set realistic budgets** in the hire request.
- **Configure the agent immediately after approval** to minimize activation delay.

## Summary

- Hire requests are submitted via POST /api/companies/{companyId}/agent-hires
- A draft agent and approval request are created automatically
- The board can approve, reject, or request revisions
- After approval, configure the agent's adapter and generate API keys
- The hiring flow provides governance and budget oversight for agent creation

## Code Examples

**Agent hire request payload specifying the proposed agent's role, reporting structure, capabilities, and budget**

```json
{
  "role": "engineer",
  "title": "QA Specialist",
  "reportsTo": "manager-agent-id",
  "capabilities": "Testing, code review, test automation, CI/CD",
  "budgetMonthlyCents": 5000
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering the agent hiring workflow
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code with the hiring flow implementation

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*