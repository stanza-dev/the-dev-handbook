---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-selection-guide"
---

# Adapter Selection Guide

## Introduction

Choosing the right adapter for each agent is one of the most impactful decisions in a Paperclip deployment. The wrong choice can lead to unnecessary costs, security concerns, or performance bottlenecks. This lesson provides a decision framework and covers the trade-offs between local and remote adapters, multi-adapter configurations, and migration strategies.

## Key Concepts

- **Decision Framework**: A structured approach to selecting the right adapter based on deployment model, latency, security, and cost requirements
- **Local vs Remote Trade-offs**: Local adapters (lower latency, data stays on-machine) versus remote adapters (scalable, separate infrastructure)
- **Multi-Adapter Companies**: Different agents in the same company using different adapter types
- **Adapter Migration**: Changing an agent's adapter type without losing context or configuration
- **Performance Monitoring**: Comparing cost and speed metrics across adapter types

## Real World Context

A growing startup initially ran all agents with `claude_local` on a single machine. As they scaled to 20 agents, the machine became a bottleneck. They migrated their less critical agents to the `http` adapter backed by a cloud GPU cluster, kept their security-sensitive agents on `claude_local` for data isolation, and used `process` adapters for automated testing pipelines. This multi-adapter strategy balanced cost, performance, and security.

## Deep Dive

### Decision Framework

Use this decision tree when selecting an adapter:

```typescript
function selectAdapter(requirements: AgentRequirements): string {
  // Does the agent need to run on the local machine?
  if (requirements.localExecution) {
    // Does it need a specific AI provider?
    if (requirements.provider === "anthropic") return "claude_local";
    if (requirements.provider === "openai") return "codex_local";
    if (requirements.provider === "google") return "gemini_local";
    if (requirements.multiProvider) return "opencode_local";
  }

  // Does it run arbitrary commands?
  if (requirements.shellCommands) return "process";

  // Does it integrate with OpenClaw?
  if (requirements.openclawGateway) return "openclaw";

  // Default: remote HTTP endpoint
  return "http";
}
```

This pseudocode captures the key decision points. Start by determining whether local execution is required, then narrow based on provider, special requirements, or default to HTTP.

### Local vs Remote Trade-offs

The choice between local and remote adapters involves three dimensions:

```typescript
// Trade-off comparison
const localAdapters = {
  latency: "Low (no network hop)",
  security: "Data stays on-machine",
  cost: "Pay per API call, no infra cost",
  scalability: "Limited by local machine resources",
  examples: ["claude_local", "codex_local", "gemini_local", "opencode_local"]
};

const remoteAdapters = {
  latency: "Higher (network round-trip)",
  security: "Data traverses network",
  cost: "Infra cost + API cost",
  scalability: "Scale horizontally with cloud resources",
  examples: ["http", "openclaw"]
};
```

Local adapters are best when data sensitivity is high and scale requirements are modest. Remote adapters are best when you need to scale beyond a single machine or when agents are already deployed as services.

### Multi-Adapter Companies

Paperclip supports using different adapters for different agents within the same company. This is common in larger deployments:

```json
{
  "agents": [
    {
      "name": "Code Reviewer",
      "adapterType": "claude_local",
      "adapterConfig": { "model": "claude-sonnet-4-20250514" }
    },
    {
      "name": "Test Runner",
      "adapterType": "process",
      "adapterConfig": { "command": "pytest", "args": ["-v"] }
    },
    {
      "name": "Cloud Analyzer",
      "adapterType": "http",
      "adapterConfig": { "url": "https://agents.cloud/analyze" }
    }
  ]
}
```

Each agent uses the adapter that best fits its purpose. The Code Reviewer needs deep reasoning (Claude), the Test Runner needs shell execution (process), and the Cloud Analyzer runs on remote infrastructure (HTTP).

### Migrating Between Adapters

When you need to change an agent's adapter type, Paperclip preserves the agent's metadata and run history. Only the adapter configuration changes:

```typescript
// Migration: claude_local → http
// Before
const oldConfig = {
  adapterType: "claude_local",
  adapterConfig: { model: "claude-sonnet-4-20250514", cwd: "/workspace" }
};

// After
const newConfig = {
  adapterType: "http",
  adapterConfig: { url: "https://claude-proxy.internal/execute", timeoutMs: 30000 }
};
// Agent ID, name, run history, and metadata are preserved
```

The agent's identity and history remain intact. Only the adapter type and configuration change. This makes migrations low-risk and reversible via Paperclip's rollback feature.

### Performance Monitoring

Paperclip tracks execution time, token usage, and cost per adapter type. Use these metrics to compare adapter performance:

```typescript
// Example dashboard metrics
const metrics = {
  claude_local: { avgLatency: "12.3s", avgCost: "$0.034", successRate: "99.2%" },
  http: { avgLatency: "18.7s", avgCost: "$0.028", successRate: "97.8%" },
  process: { avgLatency: "2.1s", avgCost: "$0.000", successRate: "95.5%" }
};
```

These metrics help you identify which adapters are most cost-effective and reliable for your workloads. If the HTTP adapter has lower costs but worse reliability, you can make an informed trade-off decision.

## Common Pitfalls

- **Using local adapters for high-scale deployments**: Local adapters compete for the same machine's resources. Beyond a few concurrent agents, consider remote adapters.
- **Choosing adapters based only on cost**: The cheapest adapter may have unacceptable latency or reliability. Consider all three dimensions.
- **Migrating without testing the new adapter first**: Always run the new adapter configuration on a test agent before migrating production agents.

## Best Practices

- **Start local, scale remote**: Begin with local adapters for simplicity, then migrate to remote adapters as you scale.
- **Use multi-adapter configurations strategically**: Match each agent's adapter to its specific requirements rather than using one adapter for everything.
- **Monitor and compare regularly**: Review adapter-level metrics monthly to identify cost savings and performance improvements.

## Summary

- Use the decision framework to select adapters based on execution location, provider, and special requirements
- Local adapters offer lower latency and data isolation; remote adapters offer better scalability
- Multi-adapter companies assign different adapter types to different agents based on their needs
- Migration between adapters preserves agent identity and run history
- Performance monitoring provides per-adapter metrics for informed trade-off decisions

## Code Examples

**Adapter selection decision framework — start with execution location, narrow by provider, then default to HTTP for remote services**

```typescript
function selectAdapter(req: AgentRequirements): string {
  if (req.localExecution) {
    if (req.provider === "anthropic") return "claude_local";
    if (req.provider === "openai") return "codex_local";
    if (req.provider === "google") return "gemini_local";
    if (req.multiProvider) return "opencode_local";
  }
  if (req.shellCommands) return "process";
  if (req.openclawGateway) return "openclaw";
  return "http";
}
```


## Resources

- [Paperclip Adapters Overview](https://paperclip.ing/docs/adapters) — Overview of all adapter types and their capabilities to inform selection decisions
- [Paperclip Monitoring Dashboard](https://paperclip.ing/docs/monitoring) — Guide to using Paperclip's monitoring dashboard for adapter-level performance comparison

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*