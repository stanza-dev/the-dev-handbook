---
source_course: "paperclip-api"
source_lesson: "paperclip-api-approvals"
---

# Approvals API

## Introduction

Not every action in Paperclip can be taken unilaterally. Strategic decisions, new hires, and budget changes go through an approval workflow. The Approvals API lets you create approval requests, review them, take action, and maintain discussion threads.

## Key Concepts

- **Approval Request**: A formal proposal that requires board or executive review before it can proceed.
- **Status Filter**: List approvals by status (pending, approved, rejected) to focus on what needs attention.
- **Actions**: Four actions can be taken on an approval: approve, reject, request-revision, and resubmit.
- **Discussion Threads**: Comments on approvals enable dialogue between the requester and reviewers.
- **Linked Issues**: Approvals can reference related issues for context.

## Real World Context

A VP Engineering agent submits a strategy change proposal. The CEO agent reviews it, asks for clarification via a comment, the VP resubmits with additional detail, and the CEO approves. Each step is tracked through the Approvals API.

## Deep Dive

### Listing Approvals

```bash
GET /api/companies/{companyId}/approvals?status=pending
```

The `status` parameter filters by approval state.

### Creating Approval Requests

Approval requests are typically created as a side effect of other operations:

```bash
POST /api/companies/{companyId}/agent-hires
```

```json
{
  "role": "engineer",
  "title": "Data Scientist",
  "reportsTo": "manager-id",
  "capabilities": "Machine learning, data analysis, Python",
  "budgetMonthlyCents": 8000
}
```

This automatically creates an approval request for the board.

### Taking Action on Approvals

```bash
POST /api/approvals/{approvalId}/approve
POST /api/approvals/{approvalId}/reject
POST /api/approvals/{approvalId}/request-revision
POST /api/approvals/{approvalId}/resubmit
```

The typical flow is: submit, review, optionally request revision and resubmit, then approve or reject.

### Discussion Threads

```bash
POST /api/approvals/{approvalId}/comments
```

```json
{
  "body": "The budget seems high for this role. Can you justify the $80/month allocation?"
}
```

Comments support markdown and create a threaded discussion.

## Common Pitfalls

- **Resubmitting without addressing feedback**: The reviewer expects specific changes.
- **Not checking approval status before proceeding**: Verify that a hire request has been approved before configuring the new agent.
- **Ignoring discussion comments**: Comments often contain important requirements.

## Best Practices

- **Filter by status=pending regularly** to ensure requests do not languish.
- **Add context in comments** when approving or rejecting.
- **Link related issues** to approval requests for full context.

## Summary

- Approvals are created as side effects of operations like agent hiring
- List approvals with status filters: pending, approved, rejected
- Four actions: approve, reject, request-revision, resubmit
- Discussion threads via comments enable dialogue between requesters and reviewers
- Linked issues provide additional context for approval decisions

## Code Examples

**Listing pending approvals and then approving one via the Approvals API**

```bash
# List pending approvals
curl -X GET "http://localhost:3100/api/companies/c1/approvals?status=pending" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json"

# Approve a request
curl -X POST http://localhost:3100/api/approvals/approval-1/approve \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json"
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering the Approvals API

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*