---
source_course: "php-async"
source_lesson: "php-async-websocket-scaling"
---

# Scaling WebSocket Applications

## Introduction
Scaling WebSocket applications is fundamentally different from scaling traditional HTTP services. While HTTP requests are stateless and short-lived, WebSocket connections are persistent and stateful, meaning you cannot simply add more servers behind a load balancer without additional infrastructure. Understanding these challenges is essential for building real-time applications that can grow beyond a single server.

## Key Concepts
- **Horizontal Scaling**: Adding more server instances to handle additional WebSocket connections, rather than upgrading a single server's resources.
- **Sticky Sessions**: A load balancer configuration that ensures a client's WebSocket connection always routes to the same backend server, which is required because the WebSocket upgrade handshake must complete on the same server that will hold the connection.
- **Pub/Sub Backbone**: A messaging layer (typically Redis Pub/Sub) that enables all server instances to share messages, so a message sent on one server can reach clients connected to any other server.
- **Connection State**: Information about each connected client (user identity, subscribed channels, last activity) that must be stored in a shared store like Redis rather than in-memory on a single server.
## Real World Context
Consider a chat application that starts on a single server handling 1,000 concurrent users. As the application grows to 50,000 users, a single server cannot handle all those persistent connections. You need multiple servers, but a message sent by a user on Server A must reach recipients on Server B and Server C. Without a pub/sub backbone, each server is an island. This is the core challenge of scaling WebSocket applications, and solving it correctly determines whether your real-time features remain reliable at scale.

## Deep Dive
The fundamental architecture for scaled WebSocket applications uses a load balancer with sticky sessions in front of multiple WebSocket servers, all connected through a Redis Pub/Sub message bus.

### Load Balancer Configuration
The load balancer must support WebSocket upgrades and sticky sessions. Sticky sessions ensure that once a client establishes a WebSocket connection with a specific server, all subsequent communication stays with that server. Without sticky sessions, the HTTP upgrade request might go to one server while subsequent WebSocket frames go to another, breaking the connection.

### Redis Pub/Sub for Cross-Server Messaging
When a message needs to reach clients across multiple servers, each server publishes the message to a Redis channel. All servers subscribe to relevant channels and forward received messages to their locally connected clients.

```php
<?php
require 'vendor/autoload.php';

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;
use React\EventLoop\Loop;

class ScalableChatServer implements MessageComponentInterface {
    protected \SplObjectStorage $clients;
    protected array $channels = [];
    private \React\Redis\RedisClient $pubRedis;
    private \React\Redis\RedisClient $subRedis;

    public function __construct() {
        $this->clients = new \SplObjectStorage();

        // Create two Redis connections: one for publishing, one for subscribing
        $factory = new \React\Redis\Factory();
        $this->pubRedis = $factory->createLazyClient('localhost:6379');
        $this->subRedis = $factory->createLazyClient('localhost:6379');

        // Subscribe to the broadcast channel
        $this->subRedis->subscribe('chat:broadcast');
        $this->subRedis->on('message', function (string $channel, string $payload) {
            $data = json_decode($payload, true);
            $this->deliverToLocalClients($data['channel'], $data['message']);
        });
    }

    public function onMessage(ConnectionInterface $from, $msg): void {
        $data = json_decode($msg, true);

        if ($data['action'] === 'send') {
            // Publish via Redis so all servers receive it
            $this->pubRedis->publish('chat:broadcast', json_encode([
                'channel' => $data['channel'],
                'message' => $data['content'],
            ]));
        }
    }

    private function deliverToLocalClients(string $channel, string $message): void {
        foreach ($this->channels[$channel] ?? [] as $client) {
            $client->send(json_encode([
                'type' => 'message',
                'channel' => $channel,
                'content' => $message,
            ]));
        }
    }

    public function onOpen(ConnectionInterface $conn): void {
        $this->clients->attach($conn);
    }

    public function onClose(ConnectionInterface $conn): void {
        $this->clients->detach($conn);
        foreach ($this->channels as $channel => $clients) {
            $clients->detach($conn);
        }
    }

    public function onError(ConnectionInterface $conn, \Exception $e): void {
        $conn->close();
    }
}
```

### Connection State Management
Instead of storing user sessions and channel subscriptions in memory on each server, use Redis hashes to maintain a shared state:

```php
<?php
// Store connection state in Redis
$redis->hSet("ws:connections:{$serverId}", $connId, json_encode([
    'userId' => $userId,
    'channels' => $subscribedChannels,
    'connectedAt' => time(),
]));

// On disconnect, clean up
$redis->hDel("ws:connections:{$serverId}", $connId);

// Get total connected users across all servers
$totalConnections = 0;
foreach ($serverIds as $sid) {
    $totalConnections += $redis->hLen("ws:connections:{$sid}");
}
```

### Heartbeats and Dead Connection Detection
In a distributed setup, detecting dead connections becomes more important because stale connection records in Redis can lead to phantom users:

```php
<?php
// Periodic heartbeat check
Loop::addPeriodicTimer(30.0, function () use ($redis, $serverId) {
    foreach ($this->clients as $client) {
        if (time() - $client->lastPong > 60) {
            $client->close();
            $redis->hDel("ws:connections:{$serverId}", $client->resourceId);
        }
    }
});
```

## Common Pitfalls
1. **Not using sticky sessions** - WebSocket upgrade requests must be handled by the same server that will maintain the connection. Without sticky sessions, the upgrade handshake fails or the connection drops immediately after establishment.
2. **Storing state in memory only** - When a server restarts or crashes, all in-memory state (subscriptions, presence data) is lost. Clients reconnect to potentially different servers with no context about their previous session.
3. **Broadcasting to all connections from one server** - If you only broadcast to locally connected clients, users on other server instances will never receive the messages. Every broadcast must go through the pub/sub backbone.

## Best Practices
1. **Use Redis or similar for pub/sub** - Redis Pub/Sub provides low-latency, cross-server message delivery. Every message that needs to reach users on other servers must pass through this shared channel.
2. **Implement connection heartbeats** - Send periodic ping/pong frames to detect dead connections early and clean up their state from the shared store. A 30-second interval with a 10-second timeout is a common configuration.
3. **Design for reconnection** - Clients should automatically reconnect with exponential backoff (e.g., 1s, 2s, 4s, 8s up to 30s max). On reconnection, the server should restore the client's previous subscriptions from the shared state store.

## Summary
- WebSocket scaling requires sticky sessions, a pub/sub backbone, and shared state storage.
- Redis Pub/Sub enables message delivery across multiple server instances.
- Connection state must be stored externally (Redis) rather than in-memory on individual servers.
- Heartbeat mechanisms are essential for detecting and cleaning up dead connections in distributed setups.
- Client-side reconnection with exponential backoff ensures resilience when servers are added, removed, or restarted.

## Resources

- [Scaling WebSocket Applications](https://www.php.net/manual/en/book.sockets.php) — PHP sockets documentation for low-level WebSocket implementation

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*