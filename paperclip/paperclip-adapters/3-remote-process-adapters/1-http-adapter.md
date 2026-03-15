---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-http-adapter"
---

# HTTP Adapter

## Introduction

The HTTP adapter (`http`) connects Paperclip to any external service that speaks HTTP. Instead of spawning a local CLI process, it sends a webhook request to a URL you configure. This makes it ideal for connecting cloud-hosted agents, external microservices, and custom APIs that process tasks remotely.

## Key Concepts

- **Adapter Type**: `http` — sends HTTP requests to external endpoints instead of spawning local processes
- **Webhook Pattern**: Paperclip sends a POST request with task context; the external service processes it and returns results
- **Payload Template**: A JSON template with Paperclip variables that gets rendered and sent as the request body
- **Authentication Headers**: Custom headers for passing API keys, bearer tokens, or other credentials to the external service
- **Timeout Configuration**: Controls how long Paperclip waits for the external service to respond

## Real World Context

A company runs their AI agents on a GPU cluster in the cloud. The agents expose an HTTP API for receiving tasks. Instead of installing CLI tools on the Paperclip server, the team configures the `http` adapter to send task requests to the cloud API. This separation means the Paperclip orchestrator and the agent runtime can scale independently.

## Deep Dive

The HTTP adapter configuration specifies the endpoint, method, headers, timeout, and payload template:

```json
{
  "adapterType": "http",
  "adapterConfig": {
    "url": "https://agents.example.com/api/v1/execute",
    "method": "POST",
    "headers": {
      "Authorization": "Bearer {{secrets.agent_api_key}}",
      "Content-Type": "application/json",
      "X-Paperclip-Run": "{{run.id}}"
    },
    "timeoutMs": 15000,
    "payloadTemplate": {
      "agentId": "{{agent.id}}",
      "runId": "{{run.id}}",
      "taskId": "{{task.id}}",
      "prompt": "{{renderedPrompt}}",
      "workspace": "{{agent.workspace}}"
    }
  }
}
```

Let us trace the flow. When a heartbeat triggers this agent, Paperclip renders the `payloadTemplate` by replacing all `{{...}}` placeholders with actual values. The `headers` are also rendered, so `{{secrets.agent_api_key}}` is resolved from the secret manager.

The rendered payload is then sent as an HTTP POST to the configured URL:

```typescript
// Inside the http adapter's execute function
const response = await fetch(config.url, {
  method: config.method,
  headers: renderedHeaders,
  body: JSON.stringify(renderedPayload),
  signal: AbortSignal.timeout(config.timeoutMs)
});

const result = await response.json();
```

The adapter uses `AbortSignal.timeout()` to enforce the configured timeout. If the external service does not respond within `timeoutMs` milliseconds, the request is aborted and Paperclip records a timeout error.

The external service is expected to return a JSON response matching the `AdapterExecutionResult` structure:

```json
{
  "sessionId": "remote-sess-456",
  "usage": {
    "inputTokens": 2000,
    "outputTokens": 5000,
    "totalTokens": 7000,
    "costUsd": 0.045
  },
  "summary": "Implemented feature X across 4 files",
  "errors": []
}
```

Paperclip parses this response and updates its run records, exactly as it would for a local adapter.

## Common Pitfalls

- **Not setting appropriate timeouts**: The default `timeoutMs` of 15000 (15 seconds) may be too short for complex AI tasks. Adjust based on expected processing time.
- **Forgetting to handle HTTPS in development**: If your external service uses a self-signed certificate, the adapter will reject the connection. Use proper certificates or configure TLS appropriately.
- **Returning non-standard response formats**: The external service must return a JSON body matching the AdapterExecutionResult structure. Custom formats will cause parsing errors.

## Best Practices

- **Use bearer tokens for authentication**: Pass API keys via the Authorization header rather than query parameters to keep credentials out of server logs.
- **Implement idempotency on the receiving end**: Paperclip may retry failed requests. Your external service should handle duplicate run IDs gracefully.
- **Set timeouts based on your SLA**: If your agent runtime typically responds in 30 seconds, set timeoutMs to 60000 (60 seconds) to allow for variance without being wasteful.

## Summary

- The `http` adapter sends webhook requests to external services instead of spawning local processes
- Configuration includes URL, method, headers, timeout, and a payload template with Paperclip variables
- Headers and payload templates are rendered at runtime, resolving secrets and context variables
- The external service must return a JSON response matching AdapterExecutionResult
- Timeouts protect against unresponsive services; authentication should use bearer tokens in headers

## Code Examples

**HTTP adapter configuration with templated headers and payload — placeholders are resolved at runtime from Paperclip context and secrets**

```json
{
  "url": "https://agents.example.com/api/v1/execute",
  "method": "POST",
  "headers": {"Authorization": "Bearer {{secrets.agent_api_key}}"},
  "timeoutMs": 15000,
  "payloadTemplate": {"agentId": "{{agent.id}}", "runId": "{{run.id}}"}
}
```


## Resources

- [Paperclip HTTP Adapter Documentation](https://paperclip.ing/docs/adapters/http) — Official documentation for configuring the HTTP webhook adapter
- [Paperclip Adapter Configuration Reference](https://paperclip.ing/docs/adapters/configuration) — Complete configuration reference for all Paperclip adapters

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*