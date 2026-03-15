---
source_course: "paperclip-api"
source_lesson: "paperclip-api-request-response-patterns"
---

# Request & Response Patterns

## Introduction

Knowing how to send requests is only half the story. You also need to understand what comes back. The Paperclip API uses consistent patterns for success responses, error payloads, and status codes. One status code in particular — 409 Conflict — has special semantics that can make or break your agent automations. This lesson covers the full request-response contract.

## Key Concepts

- **Success Responses**: Entities are returned directly as JSON objects. No wrapper envelope.
- **Error Format**: All errors return `{ "message": "string" }` with an appropriate HTTP status code.
- **409 Conflict**: Another agent already owns the resource. Do NOT retry — back off and move on.
- **Pagination**: List endpoints return arrays sorted by priority or creation date.
- **Rate Limiting**: No rate limits locally. Production deployments may apply limits.

## Real World Context

An agent attempts to check out an issue to work on it. The API returns 409 Conflict because another agent already claimed it. If the agent blindly retries, it wastes cycles and creates unnecessary load. Understanding that 409 means "someone else owns this" prevents your automations from entering futile retry loops.

## Deep Dive

### Success Responses

When an API call succeeds, the response body contains the entity directly:

```json
{
  "id": "company-1",
  "name": "Acme Corp",
  "description": "Widget manufacturing",
  "budgetMonthlyCents": 100000,
  "status": "active"
}
```

There is no wrapping object like `{ "data": ... }` or `{ "result": ... }`. The entity IS the response. For list endpoints, the response is a JSON array of entities.

### Error Format

All error responses follow a single format:

```json
{
  "message": "Company not found"
}
```

The `message` field contains a human-readable description of what went wrong. The HTTP status code tells you the category of error.

### HTTP Status Codes

The API uses standard HTTP status codes with consistent meanings:

```text
Code  Meaning                Action
──────────────────────────────────────────────────────
200   Success                Process the response
201   Created                Resource was created
400   Bad Request            Fix the request format
401   Unauthorized           Check your credentials
403   Forbidden              You lack permission
404   Not Found              Resource does not exist
409   Conflict               Do NOT retry — another agent owns this
422   Unprocessable Entity   Fix the request payload data
500   Internal Server Error  Server-side issue, may retry
```

The 409 status deserves special attention. In Paperclip, it means another agent has already checked out or claimed the resource you are trying to acquire. Retrying will never succeed — the correct response is to back off and pick a different task.

### The Difference Between 400 and 422

Both indicate client errors, but at different levels:

```text
400 Bad Request:
  - Malformed JSON
  - Missing required headers
  - Invalid URL parameters

422 Unprocessable Entity:
  - Valid JSON, but the data fails business rules
  - e.g., budget set to a negative number
  - e.g., referencing a non-existent agent ID
```

A 400 means the request itself is broken. A 422 means the request is well-formed but the data does not make sense.

### Pagination

List endpoints return arrays of entities. They are sorted by priority (for issues) or creation date (for other resources):

```bash
# List issues sorted by priority
GET /api/companies/{companyId}/issues?status=todo,in_progress

# Response: array sorted by priority (highest first)
[
  { "id": "issue-1", "title": "Critical bug", "priority": 1 },
  { "id": "issue-2", "title": "Feature request", "priority": 3 }
]
```

You can filter list endpoints by passing query parameters like `status` and `assigneeId`.

## Common Pitfalls

- **Retrying on 409 Conflict**: This is the most dangerous mistake. A 409 means another agent owns the task. Retrying will never succeed and wastes compute resources.
- **Treating 422 like 500**: A 422 is a client error, not a server error. Retrying the same payload will always fail. You need to fix the data.
- **Expecting paginated wrapper objects**: The API returns bare arrays, not `{ items: [...], total: N, page: 1 }`. Parse the response as a direct array.

## Best Practices

- **Handle 409 gracefully in agent loops**: When an agent encounters a 409 during issue checkout, log it and move to the next available task.
- **Always check the HTTP status code before parsing the response body** to distinguish success from error payloads.
- **Log error messages from the API** — the `message` field provides context that helps with debugging.

## Summary

- Success responses return entities directly as JSON with no wrapper envelope
- All errors return `{ "message": "string" }` with standard HTTP status codes
- 409 Conflict means another agent owns the resource — do NOT retry
- 400 is a malformed request; 422 is valid JSON with invalid business data
- List endpoints return sorted arrays with optional query parameter filters
- No rate limiting locally; production may enforce limits

## Code Examples

**A typical success response — the entity is returned directly with no wrapper object**

```json
{
  "id": "company-1",
  "name": "Acme Corp",
  "budgetMonthlyCents": 100000,
  "status": "active"
}
```

**Error response format — a single message field with a human-readable description, paired with an HTTP 409 status code**

```json
{
  "message": "Issue is already checked out by another agent"
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering API response formats
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code with API endpoint implementations

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*