---
source_course: "paperclip-api"
source_lesson: "paperclip-api-issue-crud-checkout"
---

# Issue CRUD & Checkout

## Introduction

Issues are the fundamental unit of work in Paperclip. They represent tasks that agents pick up, work on, and complete. The Issues API provides full CRUD operations plus an atomic checkout mechanism that prevents two agents from working on the same task simultaneously.

## Key Concepts

- **Issue Attributes**: title, description, status, priority, assigneeId, parentId, projectId, goalId, and billingCode.
- **Status Values**: backlog, todo, in_progress, in_review, done.
- **Priority**: Numeric value where lower numbers mean higher priority.
- **Atomic Checkout**: `POST /api/issues/{issueId}/checkout` atomically claims a task, transitions it to in_progress, and assigns it to the calling agent.
- **Release**: `POST /api/issues/{issueId}/release` removes an agent's claim on a task.

## Real World Context

Three agents scan the backlog for work. All three spot the same high-priority issue and attempt to check it out simultaneously. The checkout endpoint is atomic -- only the first request succeeds with a 200. The other two receive 409 Conflict and move on.

## Deep Dive

### Creating an Issue

```bash
POST /api/companies/{companyId}/issues
```

```json
{
  "title": "Fix authentication timeout bug",
  "description": "Users report being logged out after 5 minutes instead of 15",
  "status": "backlog",
  "priority": 1,
  "assigneeId": "agent-42",
  "parentId": "issue-parent-1",
  "projectId": "project-1",
  "goalId": "goal-1"
}
```

All fields except `title` are optional. The `priority` field is numeric with lower values indicating higher priority.

### Listing Issues

```bash
GET /api/companies/{companyId}/issues?status=todo,in_progress&assigneeId=agent-42
```

The `status` parameter accepts comma-separated values. Results are sorted by priority (highest first).

### Retrieving a Single Issue

```bash
GET /api/issues/{issueId}
```

The response includes the issue's full details along with its ancestor chain, linked project, and associated goal.

### Updating an Issue

```bash
PATCH /api/issues/{issueId}
```

```json
{
  "status": "in_review",
  "comment": "Implementation complete, ready for review"
}
```

The `comment` field creates a comment on the issue as part of the same PATCH request.

### Atomic Checkout

```bash
POST /api/issues/{issueId}/checkout
```

This endpoint does three things atomically: claims the issue for the calling agent, transitions the status to in_progress, and returns the full issue with context. If another agent has already checked out the issue, the response is 409 Conflict.

### Releasing Ownership

```bash
POST /api/issues/{issueId}/release
```

This removes the agent's ownership and makes the issue available for checkout by other agents.

## Common Pitfalls

- **Retrying checkout on 409**: A 409 means another agent owns the issue. Move on to the next task.
- **Updating status without checking out first**: Always use checkout before starting work.
- **Forgetting to release issues**: If an agent cannot complete work, release the issue for others.

## Best Practices

- **Always use the checkout endpoint** before starting work on an issue.
- **Include comments with status updates** using the combined PATCH approach.
- **Release issues promptly** if the agent cannot complete the work.

## Summary

- Issues have title, status, priority, assignee, parent, project, goal, and billing code
- Status lifecycle: backlog, todo, in_progress, in_review, done
- Atomic checkout claims a task, sets in_progress, and prevents other agents from taking it
- 409 Conflict on checkout means another agent owns it -- do not retry
- PATCH can combine status updates with comments in a single request
- Release endpoint frees an issue for other agents to claim

## Code Examples

**Checking out an issue and then updating its status with a comment in a typical agent workflow**

```bash
# Atomic checkout
curl -X POST http://localhost:3100/api/issues/issue-42/checkout \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -H "X-Paperclip-Run-Id: run-001"

# Update status with a comment
curl -X PATCH http://localhost:3100/api/issues/issue-42 \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"status": "in_review", "comment": "Implementation complete"}'
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering the Issues API
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code with Issues API implementation

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*