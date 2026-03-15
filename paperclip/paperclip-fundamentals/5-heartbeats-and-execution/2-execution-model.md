---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-execution-model"
---

# The Execution Model

## Introduction

When a heartbeat fires, Paperclip needs to actually run the agent. This happens through the execution model — a pipeline that invokes the adapter, spawns a process, passes context, captures results, and records metrics. Understanding this model helps you configure agents effectively and debug issues when they arise.

## Key Concepts

- **Adapter Invocation**: Paperclip calls the adapter's execute() function to start the agent's work
- **Process Spawning**: The adapter spawns a child process (e.g., a Claude CLI session) and callbacks to the Paperclip API
- **Result Capture**: stdout, usage metrics, and cost data are collected from the execution
- **Session State**: Persisted between heartbeats via runtime.sessionParams
- **Environment Variables**: Injected into the agent's process for authentication and context

## Real World Context

When a manager at a company assigns a task to an employee, the employee gets a briefing document (context), does the work at their desk (process execution), and returns a report (result capture). The company records the time spent and resources used (metrics). Paperclip's execution model is this same workflow, automated for AI agents.

## Deep Dive

The execution flow starts when Paperclip triggers a heartbeat. The server invokes the configured adapter's execute() function, passing the full context: the agent's identity, the issue to work on, project details, goal context, and any session state from previous heartbeats.

The adapter then spawns a child process. For example, the `claude_local` adapter starts a Claude CLI session, while the `codex_local` adapter starts an OpenAI Codex session. The adapter is the bridge — it translates Paperclip's generic execution request into runtime-specific commands.

```bash
# Environment variables injected into the agent process:
export PAPERCLIP_AGENT_ID="agent-abc-123"
export PAPERCLIP_COMPANY_ID="company-xyz-789"
export PAPERCLIP_API_URL="http://localhost:3100/api"
export PAPERCLIP_API_KEY="pk_live_..."
export PAPERCLIP_RUN_ID="run-def-456"
export PAPERCLIP_TASK_ID="issue-ghi-012"
export PAPERCLIP_WAKE_REASON="scheduled"
```

These environment variables give the agent everything it needs to operate. PAPERCLIP_AGENT_ID identifies who the agent is. PAPERCLIP_API_URL tells it where to make API calls. PAPERCLIP_API_KEY authenticates those calls. PAPERCLIP_RUN_ID tags all activity for audit tracing. PAPERCLIP_TASK_ID identifies the specific issue being worked on. PAPERCLIP_WAKE_REASON explains why this heartbeat was triggered.

During execution, the agent process can call back to the Paperclip API to update issue status, leave comments, check out additional issues, or query for context. Every API call includes the X-Paperclip-Run-Id header for tracing:

```bash
# Agent calls back to Paperclip during execution
curl -X PATCH http://localhost:3100/api/issues/{issueId} \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
  -H "Content-Type: application/json" \
  -d '{ "status": "in_review" }'
```

This shows an agent updating an issue's status during a heartbeat. The environment variables are used directly in the curl command, and the run ID is passed as a header for audit tracking.

After execution completes, Paperclip captures the results. This includes stdout from the process, usage metrics (tokens consumed, API calls made), and cost data (dollars spent on the underlying model). These metrics feed into the budget tracking system.

Session state is managed through runtime.sessionParams. When an agent needs to remember something between heartbeats — like which step of a multi-step task it is on — it writes to sessionParams. The next heartbeat receives these params, allowing the agent to resume where it left off.

## Common Pitfalls

1. **Not using environment variables in agent code** — The env vars are injected for a reason. Hardcoding API URLs or keys defeats the portability and security model.
2. **Ignoring cost data** — Every execution incurs costs. If you do not monitor the cost data from result capture, you may exceed budgets without warning.

## Best Practices

1. **Use sessionParams for checkpointing** — If your agent's work spans multiple heartbeats, save progress in sessionParams so it can resume efficiently.
2. **Always include X-Paperclip-Run-Id in API callbacks** — This header creates the audit trail that connects every action to a specific heartbeat execution.

## Summary

- Paperclip invokes the adapter's execute() function, which spawns the agent process
- Seven environment variables are injected: PAPERCLIP_AGENT_ID, COMPANY_ID, API_URL, API_KEY, RUN_ID, TASK_ID, WAKE_REASON
- Agents call back to the Paperclip API during execution to update issues and fetch context
- Results include stdout, usage metrics, and cost data
- Session state persists between heartbeats via runtime.sessionParams

## Code Examples

**Environment variables injected into the agent process during heartbeat execution**

```bash
# Key environment variables during heartbeat:
# PAPERCLIP_AGENT_ID, PAPERCLIP_COMPANY_ID,
# PAPERCLIP_API_URL, PAPERCLIP_API_KEY,
# PAPERCLIP_RUN_ID, PAPERCLIP_TASK_ID,
# PAPERCLIP_WAKE_REASON
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation on the execution model and adapters

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*