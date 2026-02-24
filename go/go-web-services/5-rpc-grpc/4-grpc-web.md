---
source_course: "go-web-services"
source_lesson: "go-web-services-grpc-web"
---

# gRPC-Web & Connect

## Introduction
Browsers cannot use native gRPC because they lack full HTTP/2 control. gRPC-Web and Connect are two solutions that bridge this gap, letting web applications communicate with gRPC services.

## Key Concepts
- **gRPC-Web**: A protocol that translates gRPC calls into HTTP/1.1 compatible requests, typically via an Envoy proxy.
- **Connect (connectrpc.com)**: A modern RPC framework that natively supports gRPC, gRPC-Web, and its own Connect protocol over standard HTTP.
- **h2c**: HTTP/2 without TLS, used in development and behind load balancers that handle TLS termination.
- **gRPC-Gateway**: A reverse proxy that generates a REST API from your `.proto` service definitions.

## Real World Context
If you have a gRPC backend and need to add a web frontend, you face the browser compatibility problem. Connect is the modern solution — it lets you serve gRPC, gRPC-Web, and REST-like endpoints from the same handler, without a separate proxy.

## Deep Dive

gRPC-Web uses a proxy to translate between the browser and the gRPC server.

```
Browser (HTTP/1.1) -> Envoy Proxy -> gRPC Server (HTTP/2)
```

This requires deploying and configuring an Envoy proxy, adding operational complexity.

Connect is a modern alternative that supports all three protocols from a single handler.

```go
import "connectrpc.com/connect"

mux := http.NewServeMux()
mux.Handle(greetv1connect.NewGreetServiceHandler(&GreetServer{}))

http.ListenAndServe(":8080", h2c.NewHandler(mux, &http2.Server{}))
```

The same handler serves gRPC clients, gRPC-Web browsers, and Connect clients — no proxy required.

Choose the right approach based on your use case.

*   **Internal services:** Native gRPC for maximum performance.
*   **Public API with browser clients:** Connect or REST.
*   **Mixed:** gRPC-Gateway generates REST from proto.

## Common Pitfalls
1. **Deploying Envoy just for gRPC-Web** — If you only need browser support, Connect eliminates the proxy entirely.
2. **Assuming gRPC works in browsers** — Native gRPC requires HTTP/2 trailer support, which browsers do not provide.

## Best Practices
1. **Use Connect for new projects** — It supports all three protocols (gRPC, gRPC-Web, Connect) without infrastructure overhead.
2. **Use native gRPC for internal service-to-service calls** — It has the best performance and widest tooling support.

## Summary
- Browsers cannot use native gRPC — use gRPC-Web (with proxy) or Connect (no proxy).
- Connect supports gRPC, gRPC-Web, and its own protocol from a single handler.
- Use native gRPC for internal services and Connect for browser-facing endpoints.

## Code Examples

**A Connect RPC handler that serves gRPC, gRPC-Web, and Connect protocols from a single implementation — no proxy needed**

```go
type GreetServer struct{}

func (s *GreetServer) Greet(
    ctx context.Context,
    req *connect.Request[greetv1.GreetRequest],
) (*connect.Response[greetv1.GreetResponse], error) {
    return connect.NewResponse(&greetv1.GreetResponse{
        Greeting: "Hello, " + req.Msg.Name,
    }), nil
}
```


## Resources

- [Connect for Go — Getting Started](https://connectrpc.com/docs/go/getting-started) — Official Connect RPC documentation for Go
- [gRPC Go Quick Start](https://grpc.io/docs/languages/go/quickstart/) — Official gRPC tutorial for comparing native gRPC with web alternatives

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*