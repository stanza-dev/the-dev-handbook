---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-architecture"
---

# Action Cable Architecture

## Introduction
Action Cable integrates WebSockets seamlessly into Rails by mapping familiar Ruby concepts — connections, channels, and subscriptions — onto the WebSocket protocol. Understanding this architecture is essential before you write a single line of channel code.

## Key Concepts
- **Action Cable Server**: A Rack-compatible server that runs alongside your Rails app, managing all WebSocket connections and routing messages to the correct channels.
- **Connection**: Represents a single WebSocket link between a client and the server. Each browser tab that opens a WebSocket creates one connection. Connections handle authentication.
- **Channel**: The logical unit of work in Action Cable, similar to a controller in MVC. Each channel encapsulates a specific feature (e.g., ChatChannel, NotificationsChannel).
- **Subscription**: When a client subscribes to a channel, a subscription object is created on the server side. One connection can have multiple subscriptions.
- **Consumer**: The client-side JavaScript object that establishes and manages the WebSocket connection.
- **Stream**: A named broadcasting pipeline within a channel. Streams connect the server-side broadcasting mechanism to individual subscriptions.

## Real World Context
When you build a chat application, the Connection authenticates the user, the ChatChannel defines the behavior, and each chat room is a separate stream. A user viewing three chat rooms has one connection but three subscriptions, each streaming from a different room. This architecture keeps your real-time code organized the same way controllers organize your HTTP endpoints.

## Deep Dive
Action Cable's architecture follows a layered model. At the outermost layer, the Cable Server handles raw WebSocket connections. When a client connects, it creates a Connection object:

The Connection class is where you authenticate the WebSocket user:

```ruby
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private

    def find_verified_user
      if verified_user = User.find_by(id: cookies.encrypted[:user_id])
        verified_user
      else
        reject_unauthorized_connection
      end
    end
  end
end
```

The `identified_by` declaration establishes a connection identifier that lets you find specific connections later — for example, to disconnect a user who logs out.

Channels inherit from `ApplicationCable::Channel`, much like controllers inherit from `ApplicationController`:

```ruby
# app/channels/application_cable/channel.rb
module ApplicationCable
  class Channel < ActionCable::Channel::Base
    # Shared channel behavior goes here
  end
end
```

On the client side, the consumer connects to the server and manages subscriptions:

```javascript
// app/javascript/channels/consumer.js
import { createConsumer } from "@rails/actioncable"
export default createConsumer()
```

The `createConsumer()` call establishes a WebSocket connection to the `/cable` endpoint. Action Cable automatically handles reconnection if the connection drops.

The data flow follows this path:
1. Client consumer connects to Cable Server via WebSocket.
2. Connection authenticates and identifies the user.
3. Client subscribes to a channel, creating a Subscription.
4. The channel sets up streams for the subscription.
5. Server broadcasts data to a stream name.
6. All subscriptions listening on that stream receive the data.

## Common Pitfalls
1. **Skipping connection authentication** — An unauthenticated WebSocket connection allows anyone to subscribe to channels. Always verify the user in `Connection#connect` and call `reject_unauthorized_connection` for invalid users.
2. **Confusing channels with streams** — A channel is a class that defines behavior. A stream is a named pipeline within a channel. One channel can manage many streams (e.g., `ChatChannel` streams from `chat_room_1`, `chat_room_2`, etc.).

## Best Practices
1. **Keep channels focused** — Each channel should handle one feature. Do not combine chat, notifications, and presence into a single channel.
2. **Use `ApplicationCable::Channel` for shared logic** — Put common authorization checks or logging in the base channel class so all channels inherit them.
3. **Identify connections by user** — The `identified_by` mechanism lets you disconnect specific users, which is critical for logout and account suspension flows.

## Summary
- Action Cable maps WebSocket concepts to familiar Rails patterns: Connection (authentication), Channel (behavior), and Subscription (client link).
- The Connection class authenticates users via cookies or tokens and rejects unauthorized access.
- Channels are the equivalent of controllers for real-time features, each encapsulating one domain.
- Streams are named broadcasting pipelines that route messages from the server to the correct subscriptions.
- The client-side consumer manages the WebSocket connection and handles automatic reconnection.

## Code Examples

**The Connection class authenticates the WebSocket user — identified_by lets you reference and disconnect specific users later**

```ruby
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private

    def find_verified_user
      verified_user = User.find_by(id: cookies.encrypted[:user_id])
      verified_user || reject_unauthorized_connection
    end
  end
end
```


## Resources

- [Action Cable Overview - Server-Side Components](https://guides.rubyonrails.org/action_cable_overview.html#server-side-components) — Official guide section covering connections, channels, and streams

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*