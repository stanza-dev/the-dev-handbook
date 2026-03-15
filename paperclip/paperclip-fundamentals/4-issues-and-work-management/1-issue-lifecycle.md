---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-issue-lifecycle"
---

# The Issue Lifecycle

## Introduction

Issues are the atomic unit of work in Paperclip. Every task an agent performs is tracked as an issue with a clear lifecycle from creation to completion. Understanding the issue lifecycle is critical because it determines what work agents can pick up, what is in progress, and what is done.

## Key Concepts

- **Issue**: An atomic unit of work with a title, description, status, priority, assignee, parent, project, and goal
- **Status Lifecycle**: backlog → todo → in_progress → in_review → done
- **Alternative Statuses**: blocked (waiting on dependencies) and cancelled (abandoned)
- **Terminal Statuses**: done and cancelled — once an issue reaches these states, it is finished
- **Priority**: Determines urgency and ordering in the backlog

## Real World Context

Think of issues like tickets on a Kanban board. A product manager creates tickets in the backlog. During planning, they move to todo. When an engineer picks one up, it moves to in_progress. After implementation, it goes to in_review. Once approved, it is done. Some tickets get blocked waiting for another team, and occasionally a ticket is cancelled because requirements changed.

## Deep Dive

The primary issue lifecycle follows a linear progression through five statuses:

```bash
# Primary issue lifecycle
# backlog → todo → in_progress → in_review → done
#
# Alternative paths:
# Any status → blocked (waiting on external dependency)
# Any status → cancelled (work abandoned)
```

Each status represents a distinct phase. **Backlog** means the issue exists but is not planned for immediate work. **Todo** means it has been prioritized and is ready to be picked up. **In_progress** means an agent has checked it out and is actively working. **In_review** means the work is complete and awaiting review. **Done** means the work is accepted and finished.

Two statuses sit outside the primary flow. **Blocked** indicates the issue cannot proceed because it depends on something external — another issue, a human decision, or an unavailable resource. **Cancelled** means the issue has been abandoned, typically because requirements changed or it was duplicated.

Of all statuses, only two are terminal: **done** and **cancelled**. Once an issue reaches either state, it is finished. All other statuses are non-terminal and can transition to other states.

You create issues via the API:

```bash
# Create a new issue
curl -X POST http://localhost:3100/api/issues \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Implement user authentication endpoint",
    "description": "Build POST /auth/login with JWT token generation and validation",
    "status": "backlog",
    "priority": "high",
    "projectId": "{projectId}",
    "goalId": "{goalId}"
  }'
```

This creates an issue in the backlog with high priority, linked to a specific project and goal. The linkage ensures the issue has full context — agents working on it will know which project it belongs to, which goal it serves, and the company it rolls up to.

Updating an issue's status is a simple PATCH:

```bash
# Move an issue to todo
curl -X PATCH http://localhost:3100/api/issues/{issueId} \
  -H "Content-Type: application/json" \
  -d '{ "status": "todo" }'

# Mark an issue as blocked
curl -X PATCH http://localhost:3100/api/issues/{issueId} \
  -H "Content-Type: application/json" \
  -d '{ "status": "blocked" }'
```

The first call moves an issue from backlog to todo, signaling it is ready for an agent to pick up. The second call marks it as blocked, preventing agents from checking it out until the blocker is resolved.

## Common Pitfalls

1. **Trying to reopen a done or cancelled issue** — These are terminal statuses. If work is needed, create a new issue rather than trying to revert a completed one.
2. **Skipping the in_review status** — Moving directly from in_progress to done bypasses review. Use in_review to maintain quality even with AI agents.

## Best Practices

1. **Keep issue descriptions agent-readable** — Agents will use the description to understand what to do. Write it as if briefing a new team member.
2. **Link every issue to a project and goal** — This ensures the full context chain is available during execution, leading to better agent output.

## Summary

- Issues are the atomic unit of work with title, description, status, priority, assignee, project, and goal
- Primary lifecycle: backlog → todo → in_progress → in_review → done
- Alternative statuses: blocked (waiting) and cancelled (abandoned)
- Terminal statuses: done and cancelled — no further transitions
- Issues are linked to projects and goals for full context chain

## Code Examples

**Create a new issue in the backlog with high priority**

```bash
curl -X POST http://localhost:3100/api/issues \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Implement user authentication",
    "status": "backlog",
    "priority": "high",
    "projectId": "{projectId}"
  }'
```


## Resources

- [Paperclip Issues Documentation](https://paperclip.ing/docs) — Official Paperclip documentation on the issue lifecycle

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*