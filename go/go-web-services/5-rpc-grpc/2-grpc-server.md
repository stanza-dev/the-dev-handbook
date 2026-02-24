---
source_course: "go-web-services"
source_lesson: "go-web-services-grpc-server"
---

# Implementing gRPC Server

## Introduction
Implementing a gRPC server in Go involves generating code from your `.proto` file, embedding the unimplemented server struct for forward compatibility, and registering your implementation with the gRPC server.

## Key Concepts
- **UnimplementedServer**: A generated struct you embed in your server type. It provides default implementations for all methods, ensuring forward compatibility when new methods are added to the proto.
- **protoc code generation**: The `protoc` compiler with Go plugins generates interfaces, message types, and registration functions from `.proto` files.
- **grpc.NewServer()**: Creates a new gRPC server instance that listens on a TCP port.
- **Streaming RPCs**: Server-side streaming sends multiple responses; client-side streaming receives multiple requests.

## Real World Context
In a microservice architecture, each service implements one or more gRPC servers. Embedding `UnimplementedServer` is not optional — it prevents your service from crashing when a client calls a method you have not implemented yet, which happens during rolling deployments.

## Deep Dive

Generate Go code from your `.proto` file.

```bash
protoc --go_out=. --go-grpc_out=. greeter.proto
```

This generates message types and a service interface.

Implement the server by embedding the unimplemented server and overriding the methods you need.

```go
type server struct {
    pb.UnimplementedGreeterServer
}

func (s *server) SayHello(ctx context.Context, req *pb.HelloRequest) (*pb.HelloReply, error) {
    return &pb.HelloReply{
        Message: "Hello, " + req.Name,
    }, nil
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatal(err)
    }

    s := grpc.NewServer()
    pb.RegisterGreeterServer(s, &server{})

    log.Println("gRPC server listening on :50051")
    if err := s.Serve(lis); err != nil {
        log.Fatal(err)
    }
}
```

The `Serve` call blocks and handles incoming connections.

gRPC supports four communication patterns:

*   **Unary:** Request-response (most common).
*   **Server Streaming:** Server sends multiple responses.
*   **Client Streaming:** Client sends multiple requests.
*   **Bidirectional:** Both stream simultaneously.

## Common Pitfalls
1. **Not embedding `UnimplementedServer`** — Without it, adding a new method to the proto requires updating all servers simultaneously, or they will fail to compile.
2. **Forgetting to call `s.GracefulStop()` on shutdown** — Without graceful shutdown, in-flight RPCs are abruptly terminated.

## Best Practices
1. **Always embed `UnimplementedServer`** — This is required for forward compatibility and is enforced by the gRPC Go code generator.
2. **Use `s.GracefulStop()` for shutdown** — Just like HTTP graceful shutdown, this waits for in-flight RPCs to complete.

## Summary
- Generate Go code from `.proto` files using `protoc` with Go plugins.
- Embed `UnimplementedServer` in your implementation for forward compatibility.
- gRPC supports unary, server streaming, client streaming, and bidirectional streaming.

## Code Examples

**A server-streaming RPC that sends multiple User messages to the client — each stream.Send call delivers one message over the persistent HTTP/2 connection**

```go
func (s *server) ListUsers(req *pb.ListRequest, stream pb.Users_ListUsersServer) error {
    for _, user := range users {
        if err := stream.Send(&pb.User{Id: user.ID, Name: user.Name}); err != nil {
            return err
        }
    }
    return nil
}
```


## Resources

- [gRPC Go Quick Start](https://grpc.io/docs/languages/go/quickstart/) — Official gRPC tutorial covering server implementation in Go

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*