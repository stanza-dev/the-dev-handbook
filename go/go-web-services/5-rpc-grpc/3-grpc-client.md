---
source_course: "go-web-services"
source_lesson: "go-web-services-grpc-client"
---

# gRPC Client & Interceptors

## Introduction
gRPC clients are generated from the same `.proto` file as the server, providing type-safe method calls. Interceptors (gRPC's middleware) add cross-cutting concerns like logging, authentication, and metrics to every RPC call.

## Key Concepts
- **grpc.NewClient()**: Creates a client connection to a gRPC server, managing the underlying HTTP/2 connection.
- **Generated client**: The `protoc` compiler generates a typed client stub with methods matching the service definition.
- **Unary interceptor**: Middleware that wraps every unary RPC call — equivalent to HTTP middleware for gRPC.
- **Deadline/timeout**: A context deadline that cancels the RPC if the server does not respond in time.

## Real World Context
In production, gRPC clients need timeouts, retries, and observability. Interceptors are the standard way to add these without modifying each call site. Every gRPC client in a microservice architecture uses interceptors for logging and tracing.

## Deep Dive

Create a client connection and make typed RPC calls.

```go
func main() {
    conn, err := grpc.NewClient("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()

    client := pb.NewGreeterClient(conn)

    resp, err := client.SayHello(context.Background(), &pb.HelloRequest{Name: "World"})
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(resp.Message)
}
```

The generated client provides type-safe methods — the compiler catches mismatched request/response types.

Interceptors add middleware to every RPC call, similar to HTTP middleware.

```go
func loggingInterceptor(
    ctx context.Context,
    req any,
    info *grpc.UnaryServerInfo,
    handler grpc.UnaryHandler,
) (any, error) {
    start := time.Now()
    resp, err := handler(ctx, req)
    log.Printf("%s took %v", info.FullMethod, time.Since(start))
    return resp, err
}

s := grpc.NewServer(grpc.UnaryInterceptor(loggingInterceptor))
```

The interceptor wraps the handler, executing before and after the actual RPC method.

## Common Pitfalls
1. **Not setting deadlines on client calls** — Without a timeout, a client call can hang forever if the server is unresponsive.
2. **Using `insecure.NewCredentials()` in production** — This disables TLS. Always use proper TLS credentials for production deployments.

## Best Practices
1. **Always set a context deadline on client calls** — Use `context.WithTimeout` to prevent calls from hanging indefinitely.
2. **Use interceptors for cross-cutting concerns** — Add logging, metrics, and auth as interceptors rather than duplicating logic in every RPC method.

## Summary
- Use `grpc.NewClient()` to create connections and generated clients for type-safe RPC calls.
- Interceptors are gRPC middleware — add logging, auth, and metrics without modifying individual methods.
- Always set context deadlines on client calls to prevent indefinite hangs.

## Code Examples

**A gRPC client call with a 5-second timeout — checking for DeadlineExceeded distinguishes timeouts from other errors**

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

resp, err := client.SayHello(ctx, &pb.HelloRequest{Name: "World"})
if err != nil {
    if status.Code(err) == codes.DeadlineExceeded {
        log.Println("Timeout!")
    }
}
```


## Resources

- [gRPC Go Quick Start](https://grpc.io/docs/languages/go/quickstart/) — Official gRPC tutorial covering client setup and interceptors in Go

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*