---
source_course: "go-web-services"
source_lesson: "go-web-services-grpc-server"
---

# gRPC Server

## Generate Go Code

```bash
protoc --go_out=. --go-grpc_out=. greeter.proto
```

## Implement Server

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

## Unary vs Streaming

*   **Unary:** Request-response (most common).
*   **Server Streaming:** Server sends multiple responses.
*   **Client Streaming:** Client sends multiple requests.
*   **Bidirectional:** Both stream simultaneously.

## Code Examples

**Server Streaming**

```go
// Server streaming example
func (s *server) ListUsers(req *pb.ListRequest, stream pb.Users_ListUsersServer) error {
    for _, user := range users {
        if err := stream.Send(&pb.User{Id: user.ID, Name: user.Name}); err != nil {
            return err
        }
    }
    return nil
}
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*