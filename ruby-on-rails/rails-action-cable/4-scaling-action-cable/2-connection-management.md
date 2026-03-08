---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-connection-management"
---

# Connection Management and Limits

## Introduction
Every WebSocket connection consumes server resources — memory, file descriptors, and threads. Understanding how to manage connections, set limits, and handle disconnections is essential for keeping your Action Cable deployment stable under load.

## Key Concepts
- **Connection Limit**: The maximum number of concurrent WebSocket connections a server process can handle, constrained by file descriptors and memory.
- **File Descriptors**: Operating system handles for open connections. Each WebSocket connection uses one file descriptor. The default limit on most Linux systems is 1,024.
- **Stale Connections**: WebSocket connections that are technically open but no longer active (e.g., the user closed their laptop without cleanly disconnecting).
- **Ping/Pong Heartbeat**: Action Cable sends periodic ping frames to detect stale connections. If a client does not respond, the connection is closed.

## Real World Context
A Rails server with the default file descriptor limit of 1,024 can handle roughly 1,000 WebSocket connections before running out. If you are deploying to a container with 512MB of RAM, each WebSocket connection consumes memory for its buffer and subscription state. Without proper connection management, your server can run out of resources and crash — taking HTTP requests down with it.

## Deep Dive
Action Cable runs inside your Puma server by default. Each WebSocket connection is hijacked from Puma and managed by Action Cable's event loop. This means WebSocket connections do not consume Puma threads, but they do consume file descriptors and memory.

Configure the connection settings in your environment:

```ruby
# config/environments/production.rb
Rails.application.configure do
  # Disable Action Cable request forgery protection if using token auth
  config.action_cable.disable_request_forgery_protection = false

  # Set the mount point (default: /cable)
  config.action_cable.mount_path = '/cable'

  # Worker pool size for processing incoming messages
  config.action_cable.worker_pool_size = 4
end
```

To increase the file descriptor limit on Linux/macOS, update your system configuration:

```bash
# Check current limit
ulimit -n

# Set for current session
ulimit -n 65536

# Permanent: add to /etc/security/limits.conf
# deploy  soft  nofile  65536
# deploy  hard  nofile  65536
```

Action Cable's built-in heartbeat mechanism detects stale connections:

```ruby
# Action Cable sends a ping every 3 seconds by default
# Configure in an initializer if needed
ActionCable.server.config.connection_class = -> { ApplicationCable::Connection }
```

To proactively disconnect a specific user (e.g., on logout or account suspension):

```ruby
# Disconnect a specific user from all their connections
ActionCable.server.remote_connections.where(current_user: user).disconnect
```

This finds all connections identified by the given user and closes them. The client-side consumer will attempt to reconnect, so ensure your authentication check in `Connection#connect` will reject the disconnected user.

For monitoring connection counts, log in an initializer:

```ruby
# Log connection count periodically
Thread.new do
  loop do
    count = ActionCable.server.connections.length
    Rails.logger.info "[ActionCable] Active connections: #{count}"
    sleep 60
  end
end
```

## Common Pitfalls
1. **Ignoring file descriptor limits** — The default limit is often 1,024. With 1,000 WebSocket connections plus HTTP connections plus database connections, you hit the ceiling fast. Always increase the limit in production.
2. **Not implementing `disconnect` on logout** — If a user logs out via HTTP but their WebSocket stays connected, they continue receiving real-time updates. Always disconnect the user's WebSocket when they log out.
3. **Running Action Cable on a separate process without shared state** — If you run Action Cable as a standalone server, it must use the same database and adapter as your main app. Mismatched configuration causes subscriptions to silently fail.

## Best Practices
1. **Increase file descriptor limits in production** — Set to at least 65,536 for servers handling WebSocket connections.
2. **Disconnect users on logout and account changes** — Use `ActionCable.server.remote_connections.where(...).disconnect` to clean up stale sessions.
3. **Monitor connection counts** — Track the number of active WebSocket connections as a metric. Set alerts when approaching your capacity limits.

## Summary
- Each WebSocket connection consumes a file descriptor and memory — the default OS limit is often too low for production.
- Action Cable's heartbeat mechanism automatically detects and closes stale connections.
- Use `remote_connections.where(...).disconnect` to force-disconnect specific users on logout or suspension.
- The worker pool size controls how many threads process incoming WebSocket messages.
- Monitor active connection counts and increase file descriptor limits before deploying to production.

## Code Examples

**Disconnecting a specific user — essential for logout flows and account suspension**

```ruby
# Force-disconnect a user from all WebSocket connections
# Call this when the user logs out or is suspended
ActionCable.server.remote_connections
  .where(current_user: user)
  .disconnect
```


## Resources

- [Action Cable in Production](https://guides.rubyonrails.org/action_cable_overview.html#deployment) — Official guide on deploying Action Cable in production environments

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*