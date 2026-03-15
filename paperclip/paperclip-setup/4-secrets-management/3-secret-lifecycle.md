---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-secret-lifecycle"
---

# Secret Lifecycle & Rotation

## Introduction

Secrets are not static — they need to be rotated periodically, updated after team changes, and managed across their entire lifecycle. This lesson covers how Paperclip handles secret versioning, how agents consume version updates, and when and how to rotate secrets effectively.

## Key Concepts

- **Version history**: Every update to a secret creates a new numbered version. Previous versions are retained.
- **"latest" resolution**: Agents using `"version": "latest"` automatically receive the newest version on their next heartbeat.
- **Pinned versions**: Agents can be pinned to a specific version number for stability during critical deployments.
- **Heartbeat cycle**: The periodic check-in where agents fetch updated configuration, including resolved secret values.

## Real World Context

A company's AWS access key is compromised in a security incident. The security team generates new AWS credentials and needs every agent using the old key to switch immediately. Because all agents reference the AWS secret with `"version": "latest"`, the team updates the secret once and every agent picks up the new key on its next heartbeat — typically within seconds. No configuration changes, no redeployments, no coordination with individual agent owners.

## Deep Dive

When you create a secret, it starts at version 1. Each subsequent update increments the version.

```bash
# Create secret (version 1)
curl -X POST http://localhost:3100/api/companies/{companyId}/secrets \
  -d '{"name": "db-password", "value": "initial-pass"}'

# Update secret (creates version 2)
curl -X PUT http://localhost:3100/api/companies/{companyId}/secrets/sec_db123 \
  -d '{"value": "rotated-pass-v2"}'

# Update again (creates version 3)
curl -X PUT http://localhost:3100/api/companies/{companyId}/secrets/sec_db123 \
  -d '{"value": "rotated-pass-v3"}'
```

Agents configured with `"version": "latest"` resolve to the highest version number. This resolution happens during the agent heartbeat cycle — the periodic check-in where agents fetch their current configuration.

```json
// Agent always gets the newest password
{
  "dbPassword": {
    "type": "secret_ref",
    "secretId": "sec_db123",
    "version": "latest"
  }
}

// Agent pinned to version 2 (ignores version 3+)
{
  "dbPassword": {
    "type": "secret_ref",
    "secretId": "sec_db123",
    "version": 2
  }
}
```

Pin versions when you need stability guarantees. During a critical deployment, you might pin an agent to a known-good secret version to prevent an unrelated secret rotation from causing unexpected behavior.

Know when to rotate secrets. There are four triggers for rotation.

```text
Rotation triggers:
1. Compromise — the secret value may have been exposed
2. Offboarding — a team member with access leaves
3. Periodic — scheduled rotation (e.g., every 90 days)
4. Policy — compliance requirements mandate rotation
```

Audit which agents use which secrets before rotating. If an agent is pinned to a specific version that you plan to deprecate, you need to update that agent's configuration as well.

```bash
# List agents and their secret references (via API)
curl http://localhost:3100/api/companies/{companyId}/agents \
  | jq '.[].config | .. | select(.type? == "secret_ref")'
```

This query traverses all agent configurations and extracts every secret_ref, showing you which secrets are in use and which versions are referenced.

## Common Pitfalls

1. **Rotating a secret without checking pinned versions** — If any agent is pinned to a specific version and you expect the rotation to apply universally, those pinned agents will continue using the old value. Audit pinned references before rotating.
2. **Not rotating after offboarding** — Former team members may have seen secret values in logs, debug output, or during their work. Rotate all secrets they could have accessed.

## Best Practices

1. **Default to "latest" for most agents** — Unless you have a specific stability requirement, use "latest" so agents automatically benefit from rotations without manual updates.
2. **Establish a rotation schedule** — Do not wait for incidents. Rotate high-value secrets (cloud provider keys, database passwords) every 90 days as a baseline.

## Summary

- Every secret update creates a new version; previous versions are retained.
- Agents with "latest" pick up new versions on their next heartbeat.
- Pin versions for stability during critical deployments.
- Rotate secrets on compromise, offboarding, periodically, or for compliance.
- Audit agent configurations to find pinned version references before rotating.

## Code Examples

**Rotating a secret and auditing which agents reference it**

```bash
# Update a secret (auto-creates new version)
curl -X PUT http://localhost:3100/api/companies/{companyId}/secrets/sec_db123 \
  -d '{"value": "new-rotated-password"}'

# Audit secret usage across agents
curl http://localhost:3100/api/companies/{companyId}/agents \
  | jq '.[].config | .. | select(.type? == "secret_ref")'
```


## Resources

- [Paperclip Secret Rotation](https://paperclip.ing/docs) — Secret versioning and rotation best practices

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*