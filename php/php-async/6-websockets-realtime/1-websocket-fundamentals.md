---
source_course: "php-async"
source_lesson: "php-async-websocket-fundamentals"
---

# WebSocket Fundamentals

WebSockets enable full-duplex, persistent connections between clients and servers—essential for real-time applications.

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

## Resources

- [Ratchet Documentation](http://socketo.me/docs/) — Official Ratchet WebSocket library documentation

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*