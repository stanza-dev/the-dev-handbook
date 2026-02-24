---
source_course: "go-web-services"
source_lesson: "go-web-services-grpc-intro"
---

# Introduction to gRPC

## Introduction
gRPC is Google's high-performance RPC framework that uses Protocol Buffers for serialization and HTTP/2 for transport. It is the standard for efficient inter-service communication in microservice architectures.

## Key Concepts
- **Protocol Buffers (Protobuf)**: A binary serialization format that is smaller and faster to parse than JSON, with strict schema definitions.
- **.proto files**: Schema files that define services and message types, used to generate client and server code automatically.
- **HTTP/2 transport**: gRPC uses HTTP/2, enabling multiplexing, header compression, and bidirectional streaming.
- **Streaming**: gRPC supports four communication types: unary, server streaming, client streaming, and bidirectional streaming.

## Real World Context
When microservices communicate internally, JSON over HTTP/1.1 adds unnecessary overhead from text parsing and connection management. gRPC with Protobuf is the industry standard for internal service-to-service calls, used by Google, Netflix, and most cloud-native platforms.

## Deep Dive

gRPC advantages over REST:

*   **Performance:** Binary format is smaller and faster to parse.
*   **Type Safety:** Schema defined in `.proto` files with generated code.
*   **Streaming:** Supports bidirectional streaming natively.
*   **Code Generation:** Clients and servers generated from the schema.

Define your service contract in a `.proto` file.

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

The `protoc` compiler generates Go interfaces and types from this file. The generated code provides type-safe clients and servers.

gRPC runs over HTTP/2, supporting four communication patterns: Unary (single request, single response), Server streaming, Client streaming, and Bidirectional streaming.

## Common Pitfalls
1. **Using gRPC for browser-facing APIs** — Browsers do not support HTTP/2 for gRPC natively. You need gRPC-Web or Connect for browser clients.
2. **Ignoring backwards compatibility** — Removing or renumbering Protobuf fields breaks existing clients. Always deprecate instead of removing.

## Best Practices
1. **Use gRPC for internal services, REST for public APIs** — gRPC excels at service-to-service communication; REST is better for browser and third-party integrations.
2. **Version your .proto files** — Use package versioning (e.g., `package greet.v1`) to evolve APIs without breaking clients.

## Summary
- gRPC uses Protocol Buffers (binary) over HTTP/2 for fast, type-safe RPC.
- Define services in `.proto` files and generate client/server code automatically.
- Best suited for internal service-to-service communication, not browser-facing APIs.

## Code Examples

**Calling a gRPC service method using the generated client — type-safe request and response with automatic serialization**

```go
// Generated code usage
resp, err := client.SayHello(ctx, &pb.HelloRequest{Name: "World"})
```


## Resources

- [gRPC Go Quick Start](https://grpc.io/docs/languages/go/quickstart/) — Official gRPC tutorial for getting started with Go

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*