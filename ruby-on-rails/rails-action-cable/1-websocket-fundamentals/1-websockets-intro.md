---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-websockets-intro"
---

# Understanding WebSockets

## Introduction
WebSockets provide a persistent, full-duplex communication channel between a client and a server over a single TCP connection. Unlike traditional HTTP request-response cycles, WebSockets let the server push data to the client the instant it becomes available, making them the foundation of every real-time web feature you use daily.

## Key Concepts
- **WebSocket Protocol**: A communication protocol (RFC 6455) that upgrades an HTTP connection to a persistent, bidirectional channel. Once the handshake completes, both client and server can send messages at any time.
- **Full-Duplex Communication**: Both sides of the connection can send and receive data simultaneously, unlike HTTP where the client must initiate every exchange.
- **HTTP Upgrade Handshake**: The process by which a standard HTTP request is promoted to a WebSocket connection via the `Upgrade: websocket` header.
- **Persistent Connection**: Unlike HTTP, a WebSocket connection stays open until explicitly closed, eliminating the overhead of repeated handshakes.

## Real World Context
Every time you see a chat message appear without refreshing, a notification badge update in real time, or a collaborative document sync across tabs, WebSockets are at work. Without them, you would need to poll the server repeatedly, wasting bandwidth and introducing latency. Modern applications like Slack, Figma, and GitHub all rely on WebSocket connections for their real-time features.

## Deep Dive
Traditional HTTP follows a strict request-response pattern. The client sends a request, the server processes it and returns a response, and the connection is closed. This works well for loading pages and submitting forms, but falls short when the server needs to notify the client of new data.

The WebSocket handshake starts as a normal HTTP request with special headers:

```
GET /cable HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

The server responds with a 101 status code, confirming the protocol switch:

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

After this handshake, both parties communicate over the same TCP socket using lightweight WebSocket frames instead of full HTTP headers. This dramatically reduces overhead — a WebSocket frame header is just 2-14 bytes compared to hundreds of bytes for HTTP headers.

Here is a comparison of polling versus WebSockets in terms of server load:

- **Polling**: Client sends a request every N seconds. If 1,000 users poll every 5 seconds, that is 200 requests per second — most returning empty responses.
- **WebSockets**: 1,000 persistent connections sit idle until the server has data to push. No wasted requests, near-zero latency.

## Common Pitfalls
1. **Using WebSockets for everything** — WebSockets are ideal for real-time, bidirectional communication. For simple CRUD operations or infrequent updates, standard HTTP requests are simpler, more cacheable, and easier to debug.
2. **Ignoring connection lifecycle** — WebSocket connections can drop due to network changes, server restarts, or timeouts. Always implement reconnection logic on the client side.
3. **Confusing Server-Sent Events with WebSockets** — SSE is a simpler, unidirectional protocol (server to client only) built on HTTP. If you only need server-to-client updates, SSE may be a better fit.

## Best Practices
1. **Use WebSockets only when bidirectional or low-latency communication is required** — chat, live collaboration, gaming, and real-time dashboards are good candidates.
2. **Always implement heartbeat/ping-pong mechanisms** — these detect dead connections and trigger reconnection before users notice.
3. **Secure your WebSocket connections** — always use `wss://` (WebSocket Secure) in production, and authenticate the connection during or immediately after the handshake.

## Summary
- WebSockets provide persistent, full-duplex communication between client and server over a single TCP connection.
- The connection starts with an HTTP upgrade handshake and then switches to the lightweight WebSocket protocol.
- They eliminate the overhead and latency of polling, making them ideal for real-time features.
- WebSocket connections require careful lifecycle management, including reconnection logic and heartbeats.
- Not every feature needs WebSockets — use them when bidirectional or low-latency communication is genuinely required.

## Code Examples

**Mounting the Action Cable server in Rails routes — this is the endpoint where WebSocket connections are established**

```ruby
# config/routes.rb
Rails.application.routes.draw do
  # Mount Action Cable at the /cable endpoint
  mount ActionCable.server => '/cable'
end
```


## Resources

- [Action Cable Overview](https://guides.rubyonrails.org/action_cable_overview.html) — Official Rails guide covering all aspects of Action Cable
- [RFC 6455 - The WebSocket Protocol](https://datatracker.ietf.org/doc/html/rfc6455) — The formal specification for the WebSocket protocol

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*