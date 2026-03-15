---
source_course: "paperclip-api"
source_lesson: "paperclip-api-error-handling-best-practices"
---

# Error Handling & Best Practices

## Introduction

APIs fail. Networks drop. Concurrent agents race for the same task. Building robust integrations with the Paperclip API means handling every error code correctly and designing your code to expect failures. This lesson covers defensive programming patterns, idempotency considerations, and how to use the `X-Paperclip-Run-Id` header for end-to-end tracing.

## Key Concepts

- **409 Conflict Handling**: Back off immediately. Do not retry. Another agent owns the resource.
- **422 Unprocessable Entity**: The request body is valid JSON but violates business rules. Fix the payload before resending.
- **Idempotency**: Some mutations (like PATCH) are naturally idempotent — sending the same update twice produces the same result. POST operations (like creating issues) are not.
- **X-Paperclip-Run-Id Tracing**: Include this header on every mutation during a heartbeat cycle to create an audit trail linking API calls to specific agent runs.
- **Defensive Response Checking**: Always verify the HTTP status code before parsing the body.

## Real World Context

An agent running a heartbeat cycle needs to check out an issue, update its status, and log a cost event. If the checkout fails with 409, the agent must not proceed with the rest of the workflow for that issue. If the cost event POST fails with 422, the agent should log the error and fix the payload rather than retrying. Proper error handling turns a fragile script into a production-grade automation.

## Deep Dive

### Defensive Response Handling

Every API call should check the status code before processing the response body:

```typescript
async function checkoutIssue(issueId: string, token: string, runId: string) {
  const res = await fetch(`http://localhost:3100/api/issues/${issueId}/checkout`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
      'X-Paperclip-Run-Id': runId,
    },
  });

  if (res.status === 409) {
    console.log(`Issue ${issueId} already owned by another agent. Skipping.`);
    return null;
  }

  if (!res.ok) {
    const error = await res.json();
    throw new Error(`Checkout failed (${res.status}): ${error.message}`);
  }

  return res.json();
}
```

This function explicitly handles the 409 case by returning null instead of throwing, allowing the caller to gracefully skip to the next task. All other errors are thrown with the status code and API message for debugging.

### The 409 Conflict Pattern

In a multi-agent environment, multiple agents may attempt to check out the same issue simultaneously. Only one succeeds — the rest receive 409:

```text
Agent A: POST /issues/42/checkout → 200 OK (owns it)
Agent B: POST /issues/42/checkout → 409 Conflict (too late)
Agent C: POST /issues/42/checkout → 409 Conflict (too late)
```

The correct response to a 409 is always the same: stop trying to acquire that resource and move on. Never implement a retry loop for 409 responses.

### Handling 422 Validation Errors

A 422 response means your JSON is syntactically valid but semantically wrong:

```typescript
async function createIssue(companyId: string, data: object, token: string) {
  const res = await fetch(`http://localhost:3100/api/companies/${companyId}/issues`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(data),
  });

  if (res.status === 422) {
    const error = await res.json();
    console.error(`Validation failed: ${error.message}`);
    // Fix the payload — do not retry with the same data
    return null;
  }

  if (!res.ok) {
    const error = await res.json();
    throw new Error(`Create issue failed (${res.status}): ${error.message}`);
  }

  return res.json();
}
```

The key insight is that retrying a 422 with the same data will always fail. You must inspect the error message, fix the payload, and then resend.

### Idempotency Considerations

PATCH requests in the Paperclip API are naturally idempotent — sending the same update twice produces the same final state:

```bash
# Both calls produce the same result
PATCH /api/issues/42  {"status": "in_review"}
PATCH /api/issues/42  {"status": "in_review"}
```

POST requests are NOT idempotent. Creating the same issue twice produces two issues. Design your automations to check for existing resources before creating new ones.

### Request Tracing with X-Paperclip-Run-Id

During a heartbeat cycle, every mutation should carry the run ID:

```typescript
const headers = {
  'Authorization': `Bearer ${jwt}`,
  'Content-Type': 'application/json',
  'X-Paperclip-Run-Id': currentRunId,
};

// All mutations in this cycle use the same headers
await fetch('/api/issues/42/checkout', { method: 'POST', headers });
await fetch('/api/issues/42', { method: 'PATCH', headers, body: '...' });
await fetch('/api/companies/c1/cost-events', { method: 'POST', headers, body: '...' });
```

This creates a complete trace of every action the agent took during a single run, making debugging and auditing straightforward.

## Common Pitfalls

- **Retrying 409 responses**: This is the single most common mistake. A 409 means the resource is owned. Retrying wastes cycles and will never succeed.
- **Ignoring error messages**: The `message` field in error responses often contains specific details about what went wrong. Always log it.
- **Not checking response status before parsing**: Attempting to parse an error response as a success entity causes confusing downstream errors.

## Best Practices

- **Build a status-code-aware HTTP wrapper** that handles 409, 422, and other codes consistently across all API calls.
- **Use X-Paperclip-Run-Id on every mutation** during heartbeat cycles for complete audit trails.
- **Check for existing resources before POST** to avoid accidental duplicates in non-idempotent operations.

## Summary

- Always check HTTP status codes before processing response bodies
- 409 Conflict means another agent owns the resource — back off, do not retry
- 422 Unprocessable Entity means the payload violates business rules — fix the data, do not retry blindly
- PATCH is idempotent; POST is not — check for existing resources before creating
- Include X-Paperclip-Run-Id on all mutations during heartbeat runs for traceability
- Defensive programming turns fragile scripts into production-grade automations

## Code Examples

**Defensive checkout function that handles 409 Conflict gracefully by returning null instead of retrying**

```typescript
async function checkoutIssue(issueId: string, token: string, runId: string) {
  const res = await fetch(`http://localhost:3100/api/issues/${issueId}/checkout`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
      'X-Paperclip-Run-Id': runId,
    },
  });
  if (res.status === 409) {
    console.log(`Issue ${issueId} already owned. Skipping.`);
    return null;
  }
  if (!res.ok) {
    const error = await res.json();
    throw new Error(`Checkout failed (${res.status}): ${error.message}`);
  }
  return res.json();
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation with error handling guidance
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code including API error handling

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*