---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-authenticated-modes"
---

# Authenticated Modes

## Introduction

When you need multiple users or remote access, Paperclip offers two authenticated modes: Auth + Private and Auth + Public. Both require login via Better Auth but differ in network exposure and security posture. Choosing the right one depends on whether your users are on a private network or the open internet.

## Key Concepts

- **Auth + Private**: Requires login, binds all network interfaces, designed for private networks like Tailscale or office LANs.
- **Auth + Public**: Requires login, expects an explicit public URL, designed for cloud deployments accessible from the internet.
- **Better Auth**: The authentication library Paperclip uses for session-based login.
- **Auto base URL detection**: In Auth + Private mode, Paperclip automatically detects its accessible URL from the network interface.

## Real World Context

A small startup team of five developers wants to share a single Paperclip instance. They use Tailscale to create a private mesh network between their laptops. Auth + Private mode is perfect — it requires login so each developer has their own identity, but it does not need a public URL since Tailscale handles private networking. If the same team later deploys Paperclip to a cloud server for external contractors, they switch to Auth + Public, which adds explicit URL configuration and additional security validations.

## Deep Dive

Switch from local trusted to an authenticated mode using the configure command.

```bash
# Switch to authenticated + private mode
pnpm paperclipai configure --section auth
```

The configure wizard walks through the setup. For Auth + Private mode, you configure Better Auth credentials and Paperclip binds to all network interfaces (0.0.0.0) instead of just localhost. It automatically detects the base URL from the machine's network interfaces.

Auth + Private is designed for environments where network-level access control already exists. If you are on a Tailscale network, a corporate VPN, or an isolated LAN, the network itself limits who can reach the server. Better Auth login adds identity on top of that network-level trust.

```bash
# Auth + Private: server accessible on all interfaces
# Tailscale example: accessible at http://100.x.y.z:3100
curl http://100.64.1.50:3100/api/health
# Returns: { "status": "ok" }
# But API calls require authentication
curl http://100.64.1.50:3100/api/issues
# Returns: 401 Unauthorized
```

Auth + Public mode adds additional requirements for internet-facing deployments.

```bash
# Configure for public deployment
pnpm paperclipai configure --section auth
# Select: authenticated + public
# Provide: explicit public URL (e.g., https://paperclip.mycompany.com)
```

In Auth + Public mode, you must provide an explicit base URL. Paperclip uses this for redirects, CORS configuration, and cookie domains. Auto-detection is disabled because the server's internal IP address does not match the public URL behind load balancers or reverse proxies.

Additional security validations in Auth + Public mode include stricter CORS policies, secure cookie flags (HttpOnly, Secure, SameSite), and validation that the configured base URL matches incoming request origins.

## Common Pitfalls

1. **Using Auth + Private on the public internet** — This mode trusts the network and auto-detects URLs. On the public internet, auto-detection may produce incorrect URLs and the relaxed CORS policy becomes a security risk. Use Auth + Public for internet-facing deployments.
2. **Forgetting to set the public URL in Auth + Public** — Without an explicit base URL, authentication redirects, cookies, and CORS will break. Always configure the full public URL including the protocol.

## Best Practices

1. **Use Tailscale for private team access** — Tailscale creates an encrypted mesh network with zero configuration. Combined with Auth + Private, it provides secure multi-user access without exposing Paperclip to the internet.
2. **Always use HTTPS for Auth + Public** — Internet-facing deployments must use HTTPS to protect authentication tokens in transit. Place Paperclip behind a reverse proxy with TLS termination.

## Summary

- Auth + Private: login required, binds all interfaces, auto-detects URL. Best for private networks and Tailscale.
- Auth + Public: login required, explicit public URL, stricter security. Best for cloud and internet deployments.
- Both use Better Auth for session-based authentication.
- Use `pnpm paperclipai configure --section auth` to switch modes.
- Match the mode to your network: private network = Auth + Private, internet = Auth + Public.

## Code Examples

**Configuring and verifying authenticated deployment mode**

```bash
# Switch to authenticated mode
pnpm paperclipai configure --section auth

# Verify authentication is required
curl http://100.64.1.50:3100/api/issues
# Returns: 401 Unauthorized
```


## Resources

- [Paperclip Authentication](https://paperclip.ing/docs) — Authentication setup and Better Auth integration

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*