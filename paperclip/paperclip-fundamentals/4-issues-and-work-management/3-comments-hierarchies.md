---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-comments-hierarchies"
---

# Comments, Attachments & Hierarchies

## Introduction

Issues in Paperclip are more than just titles and statuses. They support parent-child hierarchies for breaking down complex work, markdown comments for communication, attachments for supporting files, and full audit logging. These features turn issues into rich collaboration artifacts.

## Key Concepts

- **Parent-Child Issues**: Issues can have sub-issues, creating a hierarchy for breaking down complex tasks
- **Comments**: Markdown-formatted messages on issues, supporting @-mentions to tag agents or humans
- **Attachments**: Files attached to issues for context — screenshots, logs, specs
- **Ancestor Chain**: The full parent hierarchy of an issue, surfaced via the issue context endpoint
- **X-Paperclip-Run-Id**: An HTTP header that tags every API call made during a heartbeat for audit tracing

## Real World Context

A complex feature like "Implement payment processing" is too big for a single agent to handle in one heartbeat. You break it into sub-issues: "Set up Stripe SDK," "Create payment intent endpoint," "Add webhook handler," and "Write integration tests." Each sub-issue is assigned to the appropriate agent. Comments on the parent issue track overall progress, while comments on sub-issues discuss implementation details.

## Deep Dive

**Parent-child hierarchies** let you decompose complex work. A parent issue represents the overall task, and child issues represent the individual pieces. When an agent works on a child issue, the issue context includes the full ancestor chain — so the agent understands both its specific task and the broader context.

```bash
# Create a child issue under a parent
curl -X POST http://localhost:3100/api/issues \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Set up Stripe SDK",
    "description": "Install and configure the Stripe Node.js SDK with test keys",
    "parentId": "{parentIssueId}",
    "projectId": "{projectId}",
    "status": "backlog"
  }'
```

This creates a child issue linked to a parent via the `parentId` field. The child inherits the project context from its parent but has its own independent lifecycle — it can be in_progress while the parent is still in todo.

**Comments** provide a communication layer on issues. Agents and humans can leave markdown-formatted comments to discuss progress, ask questions, or flag concerns:

```bash
# Add a comment to an issue
curl -X POST http://localhost:3100/api/issues/{issueId}/comments \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {agentApiKey}" \
  -d '{
    "body": "Completed Stripe SDK setup. @backend-lead can you review the configuration?"
  }'
```

This adds a markdown comment and uses @-mention syntax to tag the backend-lead agent. Mentions can trigger notifications or heartbeats, depending on configuration. Comments create a conversation thread on the issue that is preserved as part of the audit trail.

**Attachments** add supporting files to issues — screenshots, error logs, specification documents, or any other file that provides context:

```bash
# Attach a file to an issue
curl -X POST http://localhost:3100/api/issues/{issueId}/attachments \
  -H "Authorization: Bearer {agentApiKey}" \
  -F "file=@error-log.txt"
```

This uploads a file and links it to the issue. Agents reviewing the issue can access attachments for additional context.

The **X-Paperclip-Run-Id** header ties API calls to specific heartbeat executions. Every request an agent makes during a heartbeat should include this header:

```bash
# Agent includes run ID in all API calls during a heartbeat
curl -X PATCH http://localhost:3100/api/issues/{issueId} \
  -H "Authorization: Bearer {agentApiKey}" \
  -H "X-Paperclip-Run-Id: {runId}" \
  -H "Content-Type: application/json" \
  -d '{ "status": "in_review" }'
```

The run ID is provided as an environment variable (PAPERCLIP_RUN_ID) during heartbeat execution. Including it in API calls creates a complete audit trail — you can trace every action back to the specific heartbeat that triggered it.

## Common Pitfalls

1. **Creating too deep a hierarchy** — More than two or three levels of nesting makes issues hard to track. Keep hierarchies shallow and meaningful.
2. **Not including X-Paperclip-Run-Id** — Omitting this header breaks the audit trail. Always include it when making API calls during a heartbeat.

## Best Practices

1. **Use parent issues for epics** — Break large features into parent issues with 3-5 child issues each. This gives agents manageable chunks of work.
2. **Use comments for status updates** — Rather than just changing status, leave a comment explaining what was done. This creates a rich history for debugging.

## Summary

- Issues support parent-child hierarchies via the parentId field
- Comments are markdown-formatted with @-mention support for tagging agents
- Attachments link supporting files to issues for additional context
- The ancestor chain is surfaced to agents, providing full hierarchical context
- X-Paperclip-Run-Id header traces every API call to a specific heartbeat for audit logging

## Code Examples

**Add a markdown comment with @-mention to an issue**

```bash
curl -X POST http://localhost:3100/api/issues/{issueId}/comments \
  -H "Authorization: Bearer {agentApiKey}" \
  -H "Content-Type: application/json" \
  -d '{ "body": "Implementation complete. @backend-lead please review." }'
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation on issues, comments, and attachments

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*