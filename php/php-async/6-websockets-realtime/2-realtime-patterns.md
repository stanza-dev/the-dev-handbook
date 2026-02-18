---
source_course: "php-async"
source_lesson: "php-async-realtime-patterns"
---

# Real-Time Application Patterns

Beyond basic messaging, real-time applications require careful architecture for scalability and reliability.

## Pub/Sub Pattern with Redis

```php
<?php
require 'vendor/autoload.php';

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class PubSubServer implements MessageComponentInterface {
    protected \SplObjectStorage $clients;
    protected array $subscriptions = [];  // channel => [connections]
    protected \Redis $redis;
    protected \Redis $subscriber;
    
    public function __construct() {
        $this->clients = new \SplObjectStorage();
        
        // Publisher connection
        $this->redis = new \Redis();
        $this->redis->connect('localhost', 6379);
        
        // Start Redis subscriber in background
        $this->startRedisSubscriber();
    }
    
    private function startRedisSubscriber(): void {
        // In practice, run this in a separate process
        // This is simplified for demonstration
        React\EventLoop\Loop::addPeriodicTimer(0.1, function () {
            $message = $this->redis->lpop('ws:messages');
            if ($message) {
                $data = json_decode($message, true);
                $this->broadcastToChannel($data['channel'], $data['message']);
            }
        });
    }
    
    public function onMessage(ConnectionInterface $from, $msg): void {
        $data = json_decode($msg, true);
        
        switch ($data['action']) {
            case 'subscribe':
                $this->subscribe($from, $data['channel']);
                break;
                
            case 'unsubscribe':
                $this->unsubscribe($from, $data['channel']);
                break;
                
            case 'publish':
                // Publish through Redis for cross-server support
                $this->redis->rpush('ws:messages', json_encode([
                    'channel' => $data['channel'],
                    'message' => $data['message']
                ]));
                break;
        }
    }
    
    private function subscribe(ConnectionInterface $conn, string $channel): void {
        if (!isset($this->subscriptions[$channel])) {
            $this->subscriptions[$channel] = new \SplObjectStorage();
        }
        $this->subscriptions[$channel]->attach($conn);
        
        $conn->send(json_encode([
            'type' => 'subscribed',
            'channel' => $channel
        ]));
    }
    
    private function unsubscribe(ConnectionInterface $conn, string $channel): void {
        if (isset($this->subscriptions[$channel])) {
            $this->subscriptions[$channel]->detach($conn);
        }
    }
    
    private function broadcastToChannel(string $channel, $message): void {
        if (!isset($this->subscriptions[$channel])) {
            return;
        }
        
        $payload = json_encode([
            'type' => 'message',
            'channel' => $channel,
            'data' => $message,
            'timestamp' => microtime(true)
        ]);
        
        foreach ($this->subscriptions[$channel] as $client) {
            $client->send($payload);
        }
    }
    
    // ... onOpen, onClose, onError
}
```

## Presence System

```php
<?php
class PresenceChannel {
    private array $members = [];  // channelId => [userId => userData]
    private $redis;
    
    public function join(
        string $channel,
        ConnectionInterface $conn,
        array $userData
    ): void {
        $userId = $userData['id'];
        
        // Track locally
        if (!isset($this->members[$channel])) {
            $this->members[$channel] = [];
        }
        $this->members[$channel][$userId] = [
            'connection' => $conn,
            'data' => $userData
        ];
        
        // Store in Redis for multi-server
        $this->redis->hSet(
            "presence:{$channel}",
            $userId,
            json_encode($userData)
        );
        
        // Notify others in channel
        $this->broadcast($channel, [
            'type' => 'member_joined',
            'member' => $userData
        ], $conn);
        
        // Send current members to new user
        $conn->send(json_encode([
            'type' => 'presence_state',
            'members' => $this->getMembers($channel)
        ]));
    }
    
    public function leave(string $channel, ConnectionInterface $conn): void {
        $userId = null;
        
        foreach ($this->members[$channel] ?? [] as $id => $member) {
            if ($member['connection'] === $conn) {
                $userId = $id;
                break;
            }
        }
        
        if ($userId) {
            $userData = $this->members[$channel][$userId]['data'];
            unset($this->members[$channel][$userId]);
            
            $this->redis->hDel("presence:{$channel}", $userId);
            
            $this->broadcast($channel, [
                'type' => 'member_left',
                'member' => $userData
            ]);
        }
    }
    
    public function getMembers(string $channel): array {
        $members = $this->redis->hGetAll("presence:{$channel}");
        return array_map('json_decode', $members);
    }
    
    private function broadcast(string $channel, array $data, ?ConnectionInterface $except = null): void {
        $payload = json_encode($data);
        
        foreach ($this->members[$channel] ?? [] as $member) {
            if ($member['connection'] !== $except) {
                $member['connection']->send($payload);
            }
        }
    }
}
```

## Rate Limiting

```php
<?php
class RateLimitedServer implements MessageComponentInterface {
    private array $messageCounts = [];  // resourceId => count
    private array $lastReset = [];      // resourceId => timestamp
    
    private const MAX_MESSAGES_PER_SECOND = 10;
    
    public function onMessage(ConnectionInterface $from, $msg): void {
        $id = $from->resourceId;
        $now = time();
        
        // Reset counter every second
        if (!isset($this->lastReset[$id]) || $this->lastReset[$id] < $now) {
            $this->messageCounts[$id] = 0;
            $this->lastReset[$id] = $now;
        }
        
        $this->messageCounts[$id]++;
        
        if ($this->messageCounts[$id] > self::MAX_MESSAGES_PER_SECOND) {
            $from->send(json_encode([
                'type' => 'error',
                'code' => 'RATE_LIMITED',
                'message' => 'Too many messages. Please slow down.'
            ]));
            return;
        }
        
        // Process message normally
        $this->handleMessage($from, $msg);
    }
    
    // ...
}
```

## Heartbeat/Keepalive

```php
<?php
class HeartbeatServer implements MessageComponentInterface {
    private array $lastPing = [];
    private const PING_INTERVAL = 30;  // seconds
    private const PONG_TIMEOUT = 10;   // seconds
    
    public function __construct() {
        // Periodic ping check
        React\EventLoop\Loop::addPeriodicTimer(5.0, function () {
            $this->checkConnections();
        });
    }
    
    private function checkConnections(): void {
        $now = time();
        
        foreach ($this->clients as $client) {
            $lastPing = $this->lastPing[$client->resourceId] ?? 0;
            
            // Send ping if interval passed
            if ($now - $lastPing > self::PING_INTERVAL) {
                $client->send(json_encode(['type' => 'ping']));
                $this->lastPing[$client->resourceId] = $now;
            }
            
            // Check for pong timeout
            $lastPong = $client->lastPong ?? $now;
            if ($now - $lastPong > self::PING_INTERVAL + self::PONG_TIMEOUT) {
                echo "Client {$client->resourceId} timed out\n";
                $client->close();
            }
        }
    }
    
    public function onMessage(ConnectionInterface $from, $msg): void {
        $data = json_decode($msg, true);
        
        if (($data['type'] ?? '') === 'pong') {
            $from->lastPong = time();
            return;
        }
        
        // Handle other messages...
    }
}
```

## Scaling WebSocket Servers

```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    │ (sticky sessions)│
                    └────────┬────────┘
           ┌─────────────────┼─────────────────┐
           │                 │                 │
    ┌──────▼─────┐    ┌──────▼─────┐    ┌──────▼─────┐
    │  WS Server │    │  WS Server │    │  WS Server │
    │     #1     │    │     #2     │    │     #3     │
    └──────┬─────┘    └──────┬─────┘    └──────┬─────┘
           │                 │                 │
           └─────────────────┼─────────────────┘
                             │
                    ┌────────▼────────┐
                    │   Redis Pub/Sub │
                    │   (message bus) │
                    └─────────────────┘
```

Key considerations:
- Use sticky sessions so clients stay connected to the same server
- Use Redis Pub/Sub to broadcast messages across servers
- Store presence/state in Redis for consistency
- Implement reconnection logic on clients

## Resources

- [PHP Manual - Redis Extension](https://www.php.net/manual/en/book.redis.php) — PHP Redis extension for pub/sub messaging

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*