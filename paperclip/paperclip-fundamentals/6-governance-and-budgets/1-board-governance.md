---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-board-governance"
---

# Board Governance

## Introduction

Paperclip puts humans in the loop through a board-of-directors governance model. Critical decisions — like hiring agents or changing company strategy — require approval from human board members before they take effect. This ensures AI agents never make irreversible organizational decisions autonomously.

## Key Concepts

- **Board of Directors**: A group of human users who review and approve critical decisions
- **Approval Types**: Categories of decisions that require board review — agent hiring and CEO strategy changes
- **Approval Workflow**: pending → approved/rejected/revision_requested → resubmitted
- **Discussion Threads**: Board members can discuss proposals before making a decision
- **Activity Logging**: Every governance decision is recorded in an immutable audit log

## Real World Context

A startup's board of directors does not run the company day-to-day, but they approve major decisions: hiring executives, setting strategy, authorizing large expenditures. Paperclip's governance works the same way. The AI agents handle the daily work, but humans approve structural changes that affect the company's direction.

## Deep Dive

Governance in Paperclip centers on approval workflows. When an action requires board approval, Paperclip creates a pending approval request and pauses the action until a board member responds.

The most common approval type is **agent hiring**. When you propose hiring a new agent via the API, the request does not immediately create the agent. Instead, it creates a pending approval:

```bash
# Propose hiring an agent (creates pending approval)
curl -X POST http://localhost:3100/api/companies/{companyId}/agent-hires \
  -H "Content-Type: application/json" \
  -d '{
    "name": "data-analyst",
    "role": "ic",
    "adapter": "gemini_local",
    "managerId": "{managerAgentId}",
    "monthlyBudget": 8000
  }'
```

This creates a hiring proposal that enters the governance pipeline. A board member must review and approve it before the agent is created.

The approval workflow follows a clear state machine:

```bash
# Approval workflow states:
# pending            → Initial state, awaiting board review
# approved           → Board approved, action proceeds
# rejected           → Board rejected, action blocked
# revision_requested → Board wants changes before deciding
# resubmitted        → Proposer updated and resubmitted for review
```

A board member can take three actions on a pending approval: approve it, reject it, or request revisions. If revisions are requested, the proposer updates the request and resubmits it, sending it back for another review.

**Discussion threads** allow board members to collaborate before deciding. A board member might ask questions, flag concerns, or request additional information:

```bash
# Board member adds a discussion comment
curl -X POST http://localhost:3100/api/approvals/{approvalId}/comments \
  -H "Content-Type: application/json" \
  -d '{
    "body": "What is the justification for the $80/month budget? Can we start lower?"
  }'

# Board member approves the request
curl -X POST http://localhost:3100/api/approvals/{approvalId}/approve \
  -H "Content-Type: application/json"
```

The first call adds a comment to the discussion thread. The second call approves the request, which triggers the downstream action (in this case, creating the agent).

**Activity logging** ensures every governance decision is permanently recorded. Who proposed the action, who approved or rejected it, when each step happened, and what was discussed — all of this is captured in an immutable audit log. This is critical for accountability and compliance.

The CEO strategy approval type works similarly. When the CEO agent proposes a strategic change — like pivoting the company's focus or adjusting team structure — it goes through the same governance pipeline. Board members review, discuss, and vote before the change takes effect.

## Common Pitfalls

1. **Bypassing governance by directly creating agents** — Some API endpoints may allow direct creation in certain configurations. Always use the governance-aware hiring endpoint to ensure proper oversight.
2. **Not checking for pending approvals** — If an agent hire is pending, the agent does not exist yet. Check the approval status before assuming an agent was created.

## Best Practices

1. **Respond to approvals promptly** — Pending approvals block agent creation and work. Establish an SLA for board review to keep operations flowing.
2. **Use discussion threads before deciding** — Discuss concerns in the approval thread rather than rejecting outright. Request revisions when a proposal needs adjustments.

## Summary

- Governance uses a board-of-directors model with human oversight
- Two main approval types: agent hiring and CEO strategy changes
- Workflow: pending → approved/rejected/revision_requested → resubmitted
- Discussion threads allow board members to collaborate before deciding
- Every governance action is recorded in an immutable activity log

## Code Examples

**Board member approves or rejects a governance request**

```bash
# Approve a pending request
curl -X POST http://localhost:3100/api/approvals/{approvalId}/approve

# Reject a pending request
curl -X POST http://localhost:3100/api/approvals/{approvalId}/reject
```


## Resources

- [Paperclip Governance Documentation](https://paperclip.ing/docs) — Official Paperclip documentation on board governance and approval workflows

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*