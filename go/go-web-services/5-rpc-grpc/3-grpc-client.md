---
source_course: "go-web-services"
source_lesson: "go-web-services-grpc-client"
---

# gRPC Client

```go
func main() {
    conn, err := grpc.Dial("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
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

## Interceptors (Middleware)

Add logging, auth, metrics:

```go
// Unary interceptor
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

## Code Examples

**Client with Timeout**

```go
// Client with timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

resp, err := client.SayHello(ctx, &pb.HelloRequest{Name: "World"})
if err != nil {
    if status.Code(err) == codes.DeadlineExceeded {
        log.Println("Timeout!")
    }
}
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*