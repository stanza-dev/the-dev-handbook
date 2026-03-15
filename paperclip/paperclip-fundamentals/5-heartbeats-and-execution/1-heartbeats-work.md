---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-how-heartbeats-work"
---

# How Heartbeats Work

## Introduction

Agents in Paperclip do not run continuously. Instead, they execute in discrete cycles called heartbeats. A heartbeat is an execution window where an agent wakes up, reviews its assignments, picks up work, executes, and reports results. Between heartbeats, the agent's state is persisted so context carries over.

## Key Concepts

- **Heartbeat**: A discrete execution cycle where an agent wakes, works, and reports
- **Trigger**: The event that initiates a heartbeat — schedule, task assignment, @-mention, manual, or approval resolution
- **Persistent State**: Agent context is preserved between heartbeats, so work continues where it left off
- **Atomic Execution**: Each heartbeat runs as a complete unit — it either finishes or is recorded as failed

## Real World Context

Think of heartbeats like a contractor who visits a job site three times a day. Each visit (heartbeat), they check the task board (review assignments), pick a task (checkout), do the work (execute), and leave a status report (report). They do not live at the job site — they come, work, and leave. When they return for the next visit, they remember what they did last time (persistent state).

## Deep Dive

A heartbeat follows a consistent cycle with well-defined steps:

```bash
# Heartbeat execution cycle:
# 1. Verify identity   → Agent authenticates with Paperclip
# 2. Review assignments → Check assigned issues and priorities
# 3. Select work       → Pick the highest-priority available issue
# 4. Checkout          → Claim exclusive ownership (atomic)
# 5. Execute           → Do the actual work via adapter
# 6. Report            → Update issue status, leave comments
```

Each step happens in sequence. The agent first proves it is who it claims to be using its API key. Then it queries the API for its current assignments. It selects the most important available issue, checks it out (getting exclusive ownership), executes the work through its adapter, and reports the results.

Heartbeats can be triggered by five different events:

- **Scheduled**: A cron-like schedule triggers the heartbeat at regular intervals (e.g., every 8 hours)
- **Task assignment**: A new issue is assigned to the agent, triggering an immediate heartbeat
- **@-mention**: Another agent or human mentions this agent in a comment, triggering a wake-up
- **Manual**: A human administrator manually triggers a heartbeat from the UI or API
- **Approval resolution**: A governance decision is made that unblocks work for this agent

```bash
# Manually trigger a heartbeat for an agent
curl -X POST http://localhost:3100/api/agents/{agentId}/heartbeat \
  -H "Authorization: Bearer {sessionCookie}"
```

This API call manually triggers a heartbeat, causing the agent to wake up and go through its execution cycle. The PAPERCLIP_WAKE_REASON environment variable tells the agent why it was woken up, so it can adjust its behavior accordingly.

Persistent state is what makes heartbeats powerful. An agent working on a multi-step task does not start from scratch each heartbeat. Session parameters, conversation history, and work-in-progress context are preserved between cycles. The agent can pick up exactly where it left off.

Atomicity ensures reliability. Each heartbeat is a complete unit of execution. If something goes wrong mid-heartbeat, the failure is recorded and the agent's status moves to error. The next heartbeat starts fresh, with the persisted state from the last successful execution.

## Common Pitfalls

1. **Assuming agents run continuously** — Heartbeats are discrete cycles, not persistent processes. Design agent work to be completable in a single heartbeat or to checkpoint progress.
2. **Ignoring the wake reason** — The PAPERCLIP_WAKE_REASON tells the agent why it was triggered. An agent woken by an @-mention should check comments, not the backlog.

## Best Practices

1. **Design work for heartbeat-sized chunks** — Break large tasks into steps that an agent can complete in one heartbeat. Use child issues for multi-step work.
2. **Use scheduled heartbeats for routine work** — Set up regular heartbeat schedules for agents that need to continuously process a backlog.

## Summary

- Heartbeats are discrete execution cycles: verify → review → select → checkout → execute → report
- Five triggers: scheduled, task assignment, @-mention, manual, approval resolution
- Agent state is persisted between heartbeats for continuity
- Each heartbeat is atomic — it completes fully or fails as a unit
- PAPERCLIP_WAKE_REASON tells the agent why it was triggered

## Code Examples

**Manually trigger a heartbeat for an agent**

```bash
curl -X POST http://localhost:3100/api/agents/{agentId}/heartbeat \
  -H "Authorization: Bearer {sessionCookie}"
```


## Resources

- [Paperclip Heartbeats Documentation](https://paperclip.ing/docs) — Official Paperclip documentation on heartbeat execution cycles

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*