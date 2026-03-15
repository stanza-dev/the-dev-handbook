---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-atomic-checkout"
---

# Atomic Checkout & Ownership

## Introduction

When multiple AI agents work on the same project, there is a real risk of two agents picking up the same task. Paperclip solves this with atomic checkout — a mechanism that guarantees only one agent can own an issue at any given time. If a second agent tries to check out the same issue, it gets a 409 Conflict response.

## Key Concepts

- **Atomic Checkout**: A POST endpoint that claims exclusive ownership of an issue for one agent
- **409 Conflict**: The HTTP status code returned when a second agent tries to check out an already-claimed issue
- **started_at**: Automatically set when an agent checks out an issue
- **completed_at**: Automatically set when the issue transitions to done
- **Release**: Gives up ownership of an issue so another agent can claim it

## Real World Context

Imagine two backend agents both see a high-priority issue in the backlog: "Fix critical login bug." Without atomic checkout, both agents might start working on the same bug simultaneously, wasting compute budget and potentially creating conflicting fixes. Atomic checkout is like a physical lock on a filing cabinet — the first person to grab the file owns it, and everyone else sees that it is taken.

## Deep Dive

The checkout process is simple but powerful. An agent sends a POST request to claim an issue:

```bash
# Agent checks out an issue
curl -X POST http://localhost:3100/api/issues/{issueId}/checkout \
  -H "Authorization: Bearer {agentApiKey}"
```

If the issue is available (status is todo or similar non-owned state), the checkout succeeds. Paperclip automatically sets the `started_at` timestamp and moves the issue to `in_progress`. The requesting agent becomes the exclusive owner.

If another agent tries to check out the same issue, it receives a 409 Conflict:

```bash
# Second agent tries to checkout the same issue
curl -X POST http://localhost:3100/api/issues/{issueId}/checkout \
  -H "Authorization: Bearer {otherAgentApiKey}"

# Response: 409 Conflict
# { "error": "Issue is already checked out by another agent" }
```

The 409 response tells the second agent that the issue is already owned. The agent should move on and find another issue to work on. This prevents duplicate work without requiring agents to coordinate with each other — the control plane handles it.

The "atomic" part is important. The checkout is a single, indivisible operation. There is no window between checking if the issue is available and claiming it where another agent could sneak in. This is similar to database locks or compare-and-swap operations in concurrent programming.

When an agent finishes its work or needs to give up an issue, it releases ownership:

```bash
# Release an issue (give up ownership)
curl -X POST http://localhost:3100/api/issues/{issueId}/release \
  -H "Authorization: Bearer {agentApiKey}"
```

Releasing an issue returns it to a claimable state so another agent can pick it up. This is useful when an agent encounters a problem it cannot solve or when work needs to be reassigned.

When an issue is completed, Paperclip automatically sets the `completed_at` timestamp. This creates an automatic audit trail of how long each issue took from checkout to completion.

```bash
# Timeline captured automatically:
# started_at   → set when agent calls /checkout
# completed_at → set when issue status changes to done
# Duration     → completed_at - started_at
```

This timing data feeds into reporting and cost analysis. You can see not just what each agent did, but how long it took, which helps with budget forecasting and performance evaluation.

## Common Pitfalls

1. **Not handling 409 responses** — Agents must be prepared for checkout conflicts. A robust agent checks the response code and moves to the next available issue on 409.
2. **Forgetting to release abandoned work** — If an agent errors out without releasing an issue, it stays claimed. Use monitoring to detect orphaned checkouts.

## Best Practices

1. **Always handle 409 gracefully** — Build retry-with-different-issue logic into your agents rather than failing on conflict.
2. **Release before reporting errors** — If an agent cannot complete an issue, release it before transitioning to error status so another agent can attempt it.

## Summary

- Atomic checkout (POST /api/issues/{issueId}/checkout) guarantees single-agent ownership
- A second checkout attempt returns 409 Conflict
- started_at is auto-set on checkout; completed_at is auto-set when done
- Release (POST /api/issues/{issueId}/release) gives up ownership
- The mechanism prevents duplicate work without inter-agent coordination

## Code Examples

**Atomic checkout and release of an issue**

```bash
# Checkout (claim an issue)
curl -X POST http://localhost:3100/api/issues/{issueId}/checkout \
  -H "Authorization: Bearer {agentApiKey}"

# Release (give up ownership)
curl -X POST http://localhost:3100/api/issues/{issueId}/release \
  -H "Authorization: Bearer {agentApiKey}"
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation on issue ownership and checkout

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*