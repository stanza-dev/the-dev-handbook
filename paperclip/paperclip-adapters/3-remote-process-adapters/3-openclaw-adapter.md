---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-openclaw-adapter"
---

# OpenClaw Adapter

## Introduction

The OpenClaw adapter (`openclaw`) connects Paperclip to OpenClaw agents via webhooks. Unlike the local adapters that spawn CLI processes, the OpenClaw adapter communicates over HTTP with an OpenClaw gateway. This integration allows Paperclip to orchestrate agents that are managed and run by OpenClaw's self-hosted messaging gateway.

## Key Concepts

- **Adapter Type**: `openclaw` — uses webhook-based communication instead of local process spawning
- **OpenClaw Gateway**: The self-hosted messaging gateway that runs OpenClaw agents
- **Webhook Invocation**: Paperclip sends task payloads to the OpenClaw gateway, which dispatches them to the appropriate agent
- **Docker Setup**: OpenClaw is commonly deployed via Docker for isolated, reproducible environments
- **Bidirectional Integration**: Paperclip orchestrates when and what to run; OpenClaw handles how the agent executes

## Real World Context

A team already uses OpenClaw to manage AI agents that respond to messages on Slack and Discord. They want to add Paperclip's orchestration capabilities — heartbeats, task scheduling, cost tracking — without replacing their OpenClaw setup. The `openclaw` adapter bridges the two systems: Paperclip handles orchestration and OpenClaw handles agent execution.

## Deep Dive

The OpenClaw adapter configuration points to an OpenClaw gateway endpoint:

```json
{
  "adapterType": "openclaw",
  "adapterConfig": {
    "gatewayUrl": "http://localhost:18789",
    "agentId": "coder-agent",
    "apiKey": {
      "type": "secret_ref",
      "secretId": "openclaw-api-key",
      "version": "latest"
    },
    "timeoutMs": 30000
  }
}
```

When Paperclip triggers a run for an agent using this adapter, the adapter sends a webhook to the OpenClaw gateway. The gateway dispatches the task to the specified agent and returns the results.

Setting up OpenClaw for use with Paperclip typically involves Docker:

```bash
# Pull and start OpenClaw via Docker
docker run -d \
  --name openclaw-gateway \
  -p 18789:18789 \
  -v ~/.openclaw:/root/.openclaw \
  openclaw/gateway:latest

# Run the onboarding wizard
docker exec -it openclaw-gateway openclaw onboard
```

The Docker setup creates an isolated environment for the OpenClaw gateway. Port 18789 is exposed so Paperclip can reach the webhook endpoint. The volume mount persists configuration and agent data across container restarts.

Once OpenClaw is running, the adapter's execution flow looks like this:

```typescript
// Inside the openclaw adapter's execute function
const response = await fetch(`${config.gatewayUrl}/api/v1/agents/${config.agentId}/execute`, {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${apiKey}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    runId: ctx.runId,
    taskId: ctx.taskId,
    prompt: renderedPrompt,
    context: ctx.agent
  }),
  signal: AbortSignal.timeout(config.timeoutMs)
});
```

The adapter sends the run context to the OpenClaw gateway's agent execution endpoint. OpenClaw handles routing to the correct agent, managing sessions, and executing the task. The response follows the standard AdapterExecutionResult format.

## Common Pitfalls

- **Forgetting to start the OpenClaw gateway**: The adapter communicates over HTTP. If the gateway is not running, all requests fail with connection errors.
- **Port conflicts with Docker**: If port 18789 is already in use, Docker will fail to bind. Check for conflicts before starting the container.
- **Not configuring the OpenClaw agent before referencing it**: The `agentId` in the adapter config must match an agent configured in OpenClaw. Create the agent in OpenClaw first.

## Best Practices

- **Run OpenClaw on the same network as Paperclip**: Minimize latency by keeping both services on the same machine or local network.
- **Use Docker volumes for persistence**: Mount the OpenClaw config directory as a volume to preserve agent configurations across container restarts.
- **Monitor both systems**: Since Paperclip and OpenClaw are separate processes, monitor both to catch issues quickly.

## Summary

- The `openclaw` adapter connects Paperclip to OpenClaw agents via HTTP webhooks
- OpenClaw is commonly deployed via Docker with the gateway listening on port 18789
- Paperclip handles orchestration (scheduling, cost tracking) while OpenClaw handles agent execution
- The adapter sends task payloads to the OpenClaw gateway's agent execution endpoint
- Both systems must be running and properly configured for the integration to work

## Code Examples

**OpenClaw adapter configuration pointing to a local gateway with secure API key reference**

```json
{
  "adapterType": "openclaw",
  "adapterConfig": {
    "gatewayUrl": "http://localhost:18789",
    "agentId": "coder-agent",
    "apiKey": {
      "type": "secret_ref",
      "secretId": "openclaw-api-key",
      "version": "latest"
    },
    "timeoutMs": 30000
  }
}
```


## Resources

- [Paperclip OpenClaw Adapter Documentation](https://paperclip.ing/docs/adapters/openclaw) — Official documentation for configuring the OpenClaw adapter in Paperclip
- [OpenClaw Getting Started](https://docs.openclaw.ai/getting-started) — OpenClaw setup and onboarding guide for use with Paperclip

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*