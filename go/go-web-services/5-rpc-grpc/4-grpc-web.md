---
source_course: "go-web-services"
source_lesson: "go-web-services-grpc-web"
---

# gRPC in Browsers

Browsers don't support HTTP/2 for gRPC. Solutions:

## gRPC-Web

Use grpc-web proxy (Envoy) to translate:

```
Browser (HTTP/1.1) -> Envoy Proxy -> gRPC Server (HTTP/2)
```

## Connect (buf.build/connect)

Connect is a modern alternative that supports:
*   gRPC protocol
*   gRPC-Web protocol
*   Connect protocol (uses standard HTTP)

```go
import "connectrpc.com/connect"

// Same handler works for all protocols
mux := http.NewServeMux()
mux.Handle(greetv1connect.NewGreetServiceHandler(&GreetServer{}))

http.ListenAndServe(":8080", h2c.NewHandler(mux, &http2.Server{}))
```

## When to Use What

*   **Internal services:** Native gRPC.
*   **Public API with browser clients:** Connect or REST.
*   **Mixed:** gRPC-Gateway generates REST from proto.

## Code Examples

**Connect RPC Handler**

```go
// Connect RPC example
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


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*