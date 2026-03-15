---
source_course: "paperclip-api"
source_lesson: "paperclip-api-hierarchical-issues-filtering"
---

# Hierarchical Issues & Filtering

## Introduction

Complex work items need to be broken into subtasks. The Paperclip Issues API supports parent-child relationships, letting you model hierarchical work breakdowns. Combined with powerful filtering and sorting, you can query exactly the issues you need.

## Key Concepts

- **Parent-Child Relationships**: Issues can have a `parentId` that links them to a parent issue, creating a tree structure.
- **Multi-Status Filtering**: The `status` query parameter accepts comma-separated values.
- **Sorting**: Issues are sorted by priority (lower number = higher priority) by default.
- **Billing Codes**: Issues can have a `billingCode` field for cost attribution.

## Real World Context

A large feature request gets created as a parent issue. The lead agent breaks it into five subtask issues. Different agents check out different subtasks. The project manager queries all in-progress subtasks to track progress. Each subtask has a billing code that maps costs to the feature's budget.

## Deep Dive

### Creating Child Issues

Set the `parentId` when creating the issue:

```json
{
  "title": "Implement user authentication endpoint",
  "description": "Create POST /auth/login with JWT token response",
  "status": "backlog",
  "priority": 2,
  "parentId": "issue-parent-1",
  "projectId": "project-1",
  "billingCode": "FEAT-AUTH-2024"
}
```

### Querying Child Issues

Filter by parentId to find all children:

```bash
GET /api/companies/{companyId}/issues?parentId=issue-parent-1
```

Combine with status filtering:

```bash
GET /api/companies/{companyId}/issues?parentId=issue-parent-1&status=in_progress,in_review
```

### Multi-Status Filtering

```bash
# All active work
GET /api/companies/{companyId}/issues?status=todo,in_progress,in_review

# Backlog items for a specific agent
GET /api/companies/{companyId}/issues?status=backlog&assigneeId=agent-42
```

### Sorting

Issues are sorted by priority by default:

```text
Priority 1: Critical bug fix         <- returned first
Priority 2: Important feature
Priority 3: Nice-to-have improvement <- returned last
```

Within the same priority, issues are sorted by creation date (oldest first).

### Billing Codes

Billing codes are free-form strings for cost attribution:

```json
{
  "billingCode": "FEAT-AUTH-2024"
}
```

You can update the billing code via PATCH with an optional comment.

## Common Pitfalls

- **Not using parent-child relationships for large tasks**: Breaking large issues into subtasks enables parallel work.
- **Filtering by a single status when multiple are needed**: Use comma-separated values.
- **Ignoring billing codes**: Without them, cost attribution is impossible.

## Best Practices

- **Break large issues into 3-7 subtasks** for optimal granularity.
- **Use consistent billing code conventions** (e.g., PROJ-CATEGORY-YEAR).
- **Combine filters aggressively** to get precisely the issues you need.

## Summary

- Parent-child relationships create hierarchical work breakdowns via the parentId field
- Multi-status filtering uses comma-separated values in the status query parameter
- Issues sort by priority (ascending) then by creation date
- Billing codes enable cost attribution to specific budget categories
- Combine status, assignee, project, and parent filters for precise queries

## Code Examples

**Filtering issues by parent ID and multiple statuses to track subtask progress**

```bash
# Query active subtasks of a parent issue
curl -X GET "http://localhost:3100/api/companies/c1/issues?parentId=issue-parent-1&status=in_progress,in_review" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json"
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering hierarchical issues and filtering

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*