---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-security-best-practices"
---

# Security Best Practices

## Introduction

Paperclip agents execute code, call APIs, and interact with external services autonomously. This power requires careful security practices. This lesson covers the principles and patterns that keep your Paperclip deployment secure, from secret management to network controls.

## Key Concepts

- **secret_ref pattern**: The practice of separating configuration from secrets by using references instead of embedded values.
- **Rotation without redeployment**: Updating a secret value without modifying agent configurations or restarting services.
- **Agent output as untrusted**: Treating all agent-generated content as potentially malicious, similar to user input in web applications.
- **Network controls**: Restricting which networks and endpoints agents can access.

## Real World Context

A security audit of an AI agent platform reveals that API keys are embedded in agent configurations stored in a database. An engineer with database read access can extract every credential in the system. By migrating to the secret_ref pattern, the audit finding is resolved — the database contains only encrypted secret values and reference IDs, not raw credentials. Even a full database dump does not expose usable secrets.

## Deep Dive

The most important security practice in Paperclip is never embedding credentials directly in configuration.

```json
// BAD: Raw credential in agent config
{
  "adapter": "openai",
  "apiKey": "sk-live-abc123def456"
}

// GOOD: Secret reference in agent config
{
  "adapter": "openai",
  "apiKey": {
    "type": "secret_ref",
    "secretId": "sec_openai_key",
    "version": "latest"
  }
}
```

The secret_ref pattern provides three benefits. First, separation of concerns — configuration describes what an agent does, secrets provide the credentials to do it. Second, rotation without redeployment — update the secret value and agents pick up the new key without any config changes. Third, audit trail — secret access and version changes are logged.

Environment variable injection at runtime is another layer of protection. Instead of storing secrets in the Paperclip database, some deployments inject credentials via environment variables that are resolved when the agent process starts.

```bash
# Set secrets as environment variables
export OPENAI_API_KEY=sk-live-abc123
export AWS_ACCESS_KEY_ID=AKIA...

# Paperclip resolves env: references at runtime
```

Treat agent output as untrusted. Agents can generate code, text, and commands that may contain injection attacks — whether intentional (prompt injection) or accidental (hallucinated commands). Never execute agent output without validation, and never feed agent output directly into privileged operations.

Apply network controls to limit agent reach. Agents should only access the networks and endpoints they need.

```bash
# Configure timeouts to prevent agents from hanging
# on unresponsive external services
AGENT_TIMEOUT_MS=30000

# Restrict outbound network access where possible
# Use firewall rules or network policies
```

Set timeouts on all agent operations. An agent stuck in an infinite loop or waiting on an unresponsive API should be terminated, not allowed to run indefinitely consuming resources.

## Common Pitfalls

1. **Trusting agent output for privileged operations** — An agent that generates a shell command should not have that command executed with elevated privileges without human review. Treat agent output like untrusted user input.
2. **Skipping secret rotation after team changes** — When a team member leaves, rotate all secrets they had access to. Delayed rotation leaves a window of exposure.

## Best Practices

1. **Implement the principle of least privilege** — Each agent should have only the secrets and permissions needed for its specific role. A documentation agent does not need AWS deployment credentials.
2. **Set timeouts on all external operations** — Agents that call external APIs should have configurable timeouts to prevent resource exhaustion from hanging connections.

## Summary

- Never embed raw credentials in agent configurations — use secret_ref.
- The secret_ref pattern enables rotation without redeployment.
- Treat all agent output as untrusted, like user input in a web app.
- Apply network controls and timeouts to limit agent reach.
- Rotate secrets when team members leave or credentials may be compromised.

## Code Examples

**Using secret_ref instead of embedding raw credentials**

```typescript
// Agent config with secret_ref pattern
{
  "adapter": "openai",
  "apiKey": {
    "type": "secret_ref",
    "secretId": "sec_openai_key",
    "version": "latest"
  }
}
```


## Resources

- [Paperclip Security Guide](https://paperclip.ing/docs) — Security best practices for Paperclip deployments

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*