---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-local-trusted"
---

# Local Trusted Mode

## Introduction

Local trusted mode is Paperclip's default deployment configuration. It requires no authentication, binds only to localhost, and is designed for solo developers working on their own machine. This is the mode you get out of the box after running `npx paperclipai onboard`.

## Key Concepts

- **Local Trusted**: The default mode where no login is required and the server only accepts connections from localhost.
- **Localhost binding**: The server listens on 127.0.0.1 only, making it inaccessible from other machines on the network.
- **Zero auth overhead**: No session management, no tokens, no login page — the UI is immediately accessible.

## Real World Context

A solo developer building a personal project does not need multi-user authentication. Requiring login on a tool that only runs on their laptop adds friction without security benefit. Local trusted mode removes that friction entirely — open the browser, start working. It is the right default for exploration, prototyping, and single-developer workflows.

## Deep Dive

In local trusted mode, the Paperclip server binds exclusively to `127.0.0.1:3100`. This means only processes on the same machine can reach it.

```bash
# Default startup uses local trusted mode
pnpm paperclipai run
```

No environment variables need to be set for this mode. The server starts without any authentication middleware, so every request is treated as trusted. The web UI loads without a login screen, and the API accepts all requests without tokens.

This mode is ideal for several scenarios: evaluating Paperclip for the first time, developing and testing agent configurations locally, running personal automation tasks, and prototyping before deploying to a shared environment.

The security model is simple — if you can reach the server, you are trusted. Since the server only listens on localhost, the only way to reach it is from the same machine. This is the same security model used by tools like Prisma Studio, pgAdmin running locally, and other developer tools.

```bash
# Verify the server is only listening on localhost
curl http://localhost:3100/api/health
# Returns: { "status": "ok" }

# From another machine on the network:
curl http://192.168.1.100:3100/api/health
# Connection refused — server not listening on network interfaces
```

When you outgrow local trusted mode — for example, when a teammate needs access or you want to run Paperclip on a server — you switch to one of the authenticated modes using the configure command.

## Common Pitfalls

1. **Trying to access from another device** — Local trusted mode binds to localhost only. If you need to access Paperclip from a phone, tablet, or another computer, you must switch to an authenticated mode.
2. **Assuming local trusted is insecure** — For single-user local development, localhost-only binding provides sufficient isolation. It is not insecure; it is appropriately scoped for its use case.

## Best Practices

1. **Start with local trusted for evaluation** — Do not complicate your first experience with authentication setup. Get familiar with Paperclip's features in local trusted mode first, then add auth when you need multi-user access.
2. **Switch modes before sharing** — If a colleague needs access, switch to authenticated mode before sharing your URL. Never try to work around localhost binding with port forwarding in trusted mode.

## Summary

- Local trusted is the default mode — no authentication required.
- Server binds to localhost only, rejecting external connections.
- Ideal for solo development, evaluation, and prototyping.
- No configuration needed — works out of the box.
- Switch to authenticated mode when you need multi-user or remote access.

## Code Examples

**Starting Paperclip in local trusted mode with no auth**

```bash
# Start in local trusted mode (default)
pnpm paperclipai run

# Health check confirms server is running
curl http://localhost:3100/api/health
```


## Resources

- [Paperclip Deployment Modes](https://paperclip.ing/docs) — Overview of all deployment modes and when to use each

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*