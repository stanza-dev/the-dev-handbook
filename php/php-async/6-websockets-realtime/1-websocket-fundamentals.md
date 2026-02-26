---
source_course: "php-async"
source_lesson: "php-async-websocket-fundamentals"
---

# WebSocket Fundamentals

## Introduction
WebSockets enable full-duplex, persistent connections between clients and servers, making them essential for real-time applications like chat, live notifications, and collaborative editing. Unlike HTTP's request-response model, either side can send data at any time.
## Key Concepts
- **WebSocket Protocol**: A full-duplex communication protocol over a single TCP connection, enabling real-time bidirectional data transfer between client and server.
- **HTTP Upgrade Handshake**: The initial HTTP request that switches the protocol from HTTP to WebSocket, establishing the persistent connection.
- **Ratchet**: The most popular PHP library for building WebSocket servers, built on top of ReactPHP's event loop.
- **Rooms/Channels**: A pattern for organizing WebSocket connections into groups so messages can be broadcast to specific subsets of connected clients.
- **Connection Lifecycle**: The events of a WebSocket connection: onOpen (connected), onMessage (data received), onClose (disconnected), onError (failure).
## Real World Context
WebSockets power real-time features like live chat, collaborative editing, notifications, and live sports scores. Any feature that requires instant server-to-client communication benefits from WebSockets.

## Deep Dive
## HTTP vs WebSocket

```
HTTP (Request-Response):
Client ──request──▶ Server
Client ◀──response── Server
(connection closed)

WebSocket (Persistent, Bidirectional):
Client ◀═══════════▶ Server
(connection stays open)
(either side can send anytime)
```

## WebSocket Handshake

WebSocket connections start with an HTTP upgrade request:

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

Server responds:

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

## Ratchet WebSocket Server

Ratchet is the most popular PHP WebSocket library:

```bash
composer require cboden/ratchet
```

```php
<?php
require 'vendor/autoload.php';

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class ChatServer implements MessageComponentInterface {
    protected \SplObjectStorage $clients;
    
    public function __construct() {
        $this->clients = new \SplObjectStorage();
        echo "Chat server started\n";
    }
    
    public function onOpen(ConnectionInterface $conn): void {
        $this->clients->attach($conn);
        echo "New connection: {$conn->resourceId}\n";
        
        $conn->send(json_encode([
            'type' => 'welcome',
            'message' => 'Connected to chat server',
            'clientId' => $conn->resourceId
        ]));
    }
    
    public function onMessage(ConnectionInterface $from, $msg): void {
        $data = json_decode($msg, true);
        
        echo "Message from {$from->resourceId}: {$msg}\n";
        
        // Broadcast to all other clients
        foreach ($this->clients as $client) {
            if ($from !== $client) {
                $client->send(json_encode([
                    'type' => 'message',
                    'from' => $from->resourceId,
                    'content' => $data['content'] ?? $msg,
                    'timestamp' => time()
                ]));
            }
        }
    }
    
    public function onClose(ConnectionInterface $conn): void {
        $this->clients->detach($conn);
        echo "Connection {$conn->resourceId} disconnected\n";
        
        // Notify others
        foreach ($this->clients as $client) {
            $client->send(json_encode([
                'type' => 'userLeft',
                'clientId' => $conn->resourceId
            ]));
        }
    }
    
    public function onError(ConnectionInterface $conn, \Exception $e): void {
        echo "Error: {$e->getMessage()}\n";
        $conn->close();
    }
}

// Run the server
use Ratchet\Server\IoServer;
use Ratchet\Http\HttpServer;
use Ratchet\WebSocket\WsServer;

$server = IoServer::factory(
    new HttpServer(
        new WsServer(
            new ChatServer()
        )
    ),
    8080
);

echo "WebSocket server running on port 8080\n";
$server->run();
```

## JavaScript Client

```javascript
const ws = new WebSocket('ws://localhost:8080');

ws.onopen = () => {
    console.log('Connected');
    ws.send(JSON.stringify({ content: 'Hello, server!' }));
};

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Received:', data);
};

ws.onclose = () => {
    console.log('Disconnected');
};

ws.onerror = (error) => {
    console.error('WebSocket error:', error);
};
```

## Adding Rooms/Channels

```php
<?php
class RoomChatServer implements MessageComponentInterface {
    protected \SplObjectStorage $clients;
    protected array $rooms = [];
    
    public function __construct() {
        $this->clients = new \SplObjectStorage();
    }
    
    public function onMessage(ConnectionInterface $from, $msg): void {
        $data = json_decode($msg, true);
        
        switch ($data['action'] ?? '') {
            case 'join':
                $this->joinRoom($from, $data['room']);
                break;
                
            case 'leave':
                $this->leaveRoom($from, $data['room']);
                break;
                
            case 'message':
                $this->broadcastToRoom(
                    $data['room'],
                    $from,
                    $data['content']
                );
                break;
        }
    }
    
    private function joinRoom(ConnectionInterface $conn, string $room): void {
        if (!isset($this->rooms[$room])) {
            $this->rooms[$room] = new \SplObjectStorage();
        }
        
        $this->rooms[$room]->attach($conn);
        $this->clients[$conn]['rooms'][] = $room;
        
        $conn->send(json_encode([
            'type' => 'joined',
            'room' => $room,
            'members' => $this->rooms[$room]->count()
        ]));
    }
    
    private function leaveRoom(ConnectionInterface $conn, string $room): void {
        if (isset($this->rooms[$room])) {
            $this->rooms[$room]->detach($conn);
        }
    }
    
    private function broadcastToRoom(
        string $room,
        ConnectionInterface $from,
        string $content
    ): void {
        if (!isset($this->rooms[$room])) {
            return;
        }
        
        $message = json_encode([
            'type' => 'message',
            'room' => $room,
            'from' => $from->resourceId,
            'content' => $content,
            'timestamp' => time()
        ]);
        
        foreach ($this->rooms[$room] as $client) {
            $client->send($message);
        }
    }
    
    public function onClose(ConnectionInterface $conn): void {
        // Remove from all rooms
        foreach ($this->rooms as $room => $clients) {
            $clients->detach($conn);
        }
        $this->clients->detach($conn);
    }
    
    // ... other methods
}
```

## WebSocket with Authentication

```php
<?php
use Ratchet\ConnectionInterface;
use Ratchet\WebSocket\WsServerInterface;

class AuthenticatedServer implements MessageComponentInterface, WsServerInterface {
    public function onOpen(ConnectionInterface $conn): void {
        // Get query parameters from WebSocket URL
        $queryString = $conn->httpRequest->getUri()->getQuery();
        parse_str($queryString, $params);
        
        $token = $params['token'] ?? null;
        
        if (!$this->validateToken($token)) {
            $conn->send(json_encode(['error' => 'Unauthorized']));
            $conn->close();
            return;
        }
        
        // Store user info on connection
        $conn->user = $this->getUserFromToken($token);
        $this->clients->attach($conn);
    }
    
    private function validateToken(?string $token): bool {
        if ($token === null) {
            return false;
        }
        
        // Validate JWT or session token
        // ...
        return true;
    }
    
    private function getUserFromToken(string $token): array {
        // Decode and return user data
        return ['id' => 1, 'name' => 'User'];
    }
    
    // Required by WsServerInterface for subprotocol support
    public function getSubProtocols(): array {
        return [];
    }
}
```

## Common Pitfalls
1. **Not handling connection drops** - Network interruptions are common. Without proper onClose/onError handling, orphaned connections waste resources.
2. **Trusting WebSocket messages without validation** - All incoming messages must be validated and sanitized, just like HTTP requests.
3. **Sending too much data on open** - Flooding a new connection with historical data can cause the client to lag or crash.

## Best Practices
1. **Validate all incoming messages** - Never trust WebSocket client data. Parse JSON safely, validate types, and sanitize content.
2. **Implement authentication early** - Authenticate during the WebSocket handshake using query parameters or headers, not after the connection is open.
3. **Use rooms/channels for message routing** - Don't broadcast to all connections. Use a channel system so clients only receive relevant messages.

## Summary
- WebSockets provide persistent, bidirectional communication unlike HTTP's request-response model.
- The connection starts with an HTTP upgrade handshake, then switches to the WebSocket protocol.
- Ratchet is the most popular PHP library for WebSocket servers.
- Rooms/channels organize connections for targeted message broadcasting.
- Authentication should happen during the WebSocket handshake using tokens.

## Resources

- [Ratchet Documentation](http://socketo.me/docs/) — Official Ratchet WebSocket library documentation

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*