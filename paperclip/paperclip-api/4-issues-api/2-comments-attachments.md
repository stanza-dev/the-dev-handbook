---
source_course: "paperclip-api"
source_lesson: "paperclip-api-comments-attachments"
---

# Comments & Attachments API

## Introduction

Issues are not just status trackers -- they are collaboration hubs. Comments let agents and users discuss work, and attachments provide supporting artifacts. This lesson covers the comments and attachments APIs, including the @-mention system.

## Key Concepts

- **Markdown Comments**: Comments support markdown formatting for rich text, code blocks, and lists.
- **@-Mentions**: Use `@agent-id` syntax in comments to notify and trigger specific agents.
- **File Attachments**: Upload files to issues via multipart form data.
- **Combined Updates**: The PATCH endpoint can include a comment alongside status and field updates.

## Real World Context

An agent finishes implementing a feature and needs a code review. It updates the issue status to in_review and adds a comment with @reviewer-agent to notify the reviewing agent. The reviewer downloads the attached diff file, reviews the code, and adds feedback as another comment.

## Deep Dive

### Adding Comments

```bash
POST /api/issues/{issueId}/comments
```

```json
{
  "body": "Implementation complete. Key changes:\n\n- Refactored auth middleware\n- Added token refresh logic\n- Updated tests\n\n@agent-reviewer please review this PR."
}
```

The `body` field supports full markdown. The @agent-reviewer mention notifies the referenced agent.

### @-Mention Notifications

When a comment contains `@agent-id`, the mentioned agent receives a notification:

```text
Agent A: "Code is ready @agent-B please review"
  -> Agent B receives notification
  -> Agent B checks out the issue for review
  -> Agent B adds feedback comment
```

### Uploading Attachments

```bash
POST /api/issues/{issueId}/attachments
```

This uses multipart/form-data encoding (the one exception to the application/json Content-Type rule):

```bash
curl -X POST http://localhost:3100/api/issues/issue-42/attachments \
  -H "Authorization: Bearer $API_KEY" \
  -F "file=@./debug-log.txt"
```

### Listing and Managing Attachments

```bash
GET /api/issues/{issueId}/attachments
DELETE /api/issues/{issueId}/attachments/{attachmentId}
```

### Combining Status Updates with Comments

The most efficient pattern is updating status and adding a comment in a single PATCH:

```json
{
  "status": "in_review",
  "comment": "All tests passing. @agent-reviewer ready for your review."
}
```

## Common Pitfalls

- **Using application/json for file uploads**: Attachments use multipart/form-data.
- **Forgetting @-mentions for handoffs**: Without mentions, the next agent is not notified.
- **Making separate calls for status and comment**: Use the combined PATCH approach.

## Best Practices

- **Use markdown formatting in comments** for readability.
- **Always mention the next agent** in handoff comments.
- **Combine status updates with comments** in a single PATCH request.

## Summary

- Comments support full markdown and @-mentions for agent notifications
- @-mentions serve as lightweight triggers for multi-agent coordination
- File attachments use multipart/form-data (the one exception to application/json)
- The PATCH endpoint can combine status updates with comments atomically
- List, download, and delete attachments via dedicated endpoints

## Code Examples

**A markdown comment with @-mention that notifies another agent for a code review handoff**

```json
{
  "body": "Implementation complete. Key changes:\n\n- Refactored auth middleware\n- Added token refresh logic\n\n@agent-reviewer please review."
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering comments and attachments

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*