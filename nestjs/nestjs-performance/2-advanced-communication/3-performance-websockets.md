---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-websockets"
---

# WebSockets

## Introduction

When you need bidirectional real-time communication, WebSockets are the answer. Unlike SSE (server-to-client only), WebSockets enable full-duplex communication for chat apps, games, collaborative tools, and live trading platforms.

## Key Concepts

- **Gateway**: WebSocket endpoint handler (like a controller)
- **@WebSocketGateway()**: Decorator defining a WebSocket gateway
- **@SubscribeMessage()**: Handler for specific message types
- **Socket.IO**: Popular WebSocket library with fallbacks

## Deep Dive

### Setup

```bash
npm install @nestjs/websockets @nestjs/platform-socket.io socket.io
```

### Basic Gateway

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  MessageBody,
  WebSocketServer,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: { origin: '*' },
})
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    console.log('Client connected:', client.id);
  }

  handleDisconnect(client: Socket) {
    console.log('Client disconnected:', client.id);
  }

  @SubscribeMessage('message')
  handleMessage(@MessageBody() data: { room: string; message: string }) {
    this.server.to(data.room).emit('message', data);
    return { event: 'message', data: 'Message sent' };
  }

  @SubscribeMessage('join')
  handleJoinRoom(client: Socket, room: string) {
    client.join(room);
    return { event: 'joined', data: room };
  }
}
```

### Broadcasting

```typescript
// To all clients
this.server.emit('event', data);

// To a room
this.server.to('room-id').emit('event', data);

// To a specific client
this.server.to(client.id).emit('event', data);

// To all except sender
client.broadcast.emit('event', data);
```

### Namespaces

```typescript
@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway {
  // Only accessible at /admin namespace
}
```

### Guards and Pipes

```typescript
@WebSocketGateway()
@UseGuards(WsAuthGuard)
export class SecureGateway {
  @SubscribeMessage('secure-action')
  @UsePipes(ValidationPipe)
  handleSecure(@MessageBody() dto: SecureDto) {
    // Guarded and validated
  }
}
```

## Common Pitfalls

1. **CORS**: Configure CORS for browser connections.
2. **Authentication**: Implement WsAuthGuard for secure gateways.
3. **Memory leaks**: Untracked connections can leak memory.

## Best Practices

- Use rooms for group communication
- Implement heartbeat/ping for connection health
- Authenticate on connection
- Clean up resources on disconnect

## Summary

WebSocketGateway handles bidirectional real-time communication. Use @SubscribeMessage() for handling events and server.emit() for broadcasting. Implement connection/disconnect handlers for cleanup and authentication.

## Resources

- [WebSockets](https://docs.nestjs.com/websockets/gateways) — Official WebSockets guide

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*