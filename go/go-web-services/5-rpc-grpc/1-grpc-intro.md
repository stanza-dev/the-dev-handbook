---
source_course: "go-web-services"
source_lesson: "go-web-services-grpc-intro"
---

# gRPC vs REST

gRPC (Google Remote Procedure Call) uses **Protocol Buffers** (binary) instead of JSON (text). It is strictly typed and much faster.

## Advantages

*   **Performance:** Binary format is smaller and faster to parse.
*   **Type Safety:** Schema defined in `.proto` files.
*   **Streaming:** Supports bidirectional streaming.
*   **Code Generation:** Clients and servers generated from schema.

## .proto Files

```protobuf
syntax = "proto3";

package greet;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

gRPC runs over HTTP/2, supporting streaming (Client-side, Server-side, or Bidirectional).

## Code Examples

**gRPC Client Call**

```go
// Generated code usage
resp, err := client.SayHello(ctx, &pb.HelloRequest{Name: "World"})
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*