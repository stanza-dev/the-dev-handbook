---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-storing-secrets"
---

# Storing and Using Secrets

## Introduction

Agents in Paperclip often need credentials to interact with external services — API keys for cloud providers, database passwords, authentication tokens. Paperclip's secrets system stores these values encrypted at rest and provides a reference mechanism so agents can use secrets without ever seeing the raw values in configuration.

## Key Concepts

- **Secret**: A named, encrypted value stored in Paperclip's database. Only the ID and metadata are returned by the API — the raw value is never exposed after creation.
- **secret_ref**: A JSON object that references a secret by ID and version, used in agent configuration instead of raw credentials.
- **Auto-versioning**: Every update to a secret creates a new version. The previous versions are retained for rollback.
- **"latest" version**: A special version identifier that always resolves to the most recent version of a secret.

## Real World Context

Imagine an agent that deploys code to AWS. It needs an AWS access key. Embedding the key directly in the agent's configuration means anyone who can view the config sees the key. If the key is compromised, you must find and update every agent that uses it. With Paperclip's secret_ref pattern, the agent's config contains only a reference — the actual key is resolved at runtime from the encrypted secrets store. Rotating the key means updating one secret, and all agents pick up the new value automatically.

## Deep Dive

Create a secret using the REST API. The secret is encrypted before storage, and only the ID and metadata are returned.

```bash
# Create a secret via the API
curl -X POST http://localhost:3100/api/companies/{companyId}/secrets \
  -H "Content-Type: application/json" \
  -d '{"name": "aws-access-key", "value": "AKIA..."}'
```

The response includes the secret's ID but never the value. This is a one-way operation — once stored, the raw value cannot be retrieved through the API.

```json
{
  "id": "sec_abc123",
  "name": "aws-access-key",
  "version": 1,
  "createdAt": "2026-03-14T10:00:00Z"
}
```

To use a secret in an agent's configuration, replace the raw credential with a `secret_ref` object.

```json
{
  "adapter": "claude_local",
  "apiKey": {
    "type": "secret_ref",
    "secretId": "sec_abc123",
    "version": "latest"
  }
}
```

The `secret_ref` tells Paperclip to resolve the actual value at runtime when the agent needs it. Using `"version": "latest"` means the agent always gets the most recent version of the secret. This enables zero-downtime secret rotation — update the secret value, and agents pick up the new version on their next heartbeat cycle.

When you update a secret, a new version is created automatically.

```bash
# Update a secret (creates version 2)
curl -X PUT http://localhost:3100/api/companies/{companyId}/secrets/sec_abc123 \
  -H "Content-Type: application/json" \
  -d '{"value": "AKIA-new-key..."}'
```

Each update increments the version number. Agents configured with `"version": "latest"` will use the new value on their next heartbeat. Agents pinned to a specific version number continue using the old value until explicitly updated.

## Common Pitfalls

1. **Trying to read back a secret's value** — The API deliberately never returns the raw secret value after creation. If you lose the value, you must create a new secret. This is a security feature, not a bug.
2. **Embedding credentials directly in agent config** — Skipping the secret_ref pattern for convenience creates a security liability. Anyone with config access sees the raw credential, and rotation requires updating every agent individually.

## Best Practices

1. **Always use secret_ref instead of raw values** — Even for development, building the habit of using secret references prevents accidental credential exposure when configurations are shared or exported.
2. **Use "latest" version for credentials that may rotate** — This ensures agents automatically pick up new values without configuration changes. Pin to a specific version only when you need stability guarantees during a deployment.

## Summary

- Secrets are stored encrypted at rest — only ID and metadata are returned by the API.
- Create secrets via POST to `/api/companies/{companyId}/secrets`.
- Use `secret_ref` objects in agent config instead of raw credentials.
- Updates create new versions automatically.
- Set version to "latest" for automatic rotation pickup.

## Code Examples

**Creating an encrypted secret via the Paperclip API**

```bash
# Create a secret
curl -X POST http://localhost:3100/api/companies/{companyId}/secrets \
  -H "Content-Type: application/json" \
  -d '{"name": "aws-key", "value": "AKIA..."}'
```


## Resources

- [Paperclip Secrets Management](https://paperclip.ing/docs) — Guide to storing and referencing secrets in Paperclip

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*