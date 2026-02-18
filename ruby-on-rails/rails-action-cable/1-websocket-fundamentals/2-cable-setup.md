---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-action-cable-setup"
---

# Setting Up Action Cable

Action Cable comes built into Rails. Let's configure it for development and production.

## Configuration Files

Action Cable configuration lives in:

```yaml
# config/cable.yml
development:
  adapter: async

test:
  adapter: test

production:
  adapter: redis
  url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  channel_prefix: myapp_production
```

### Adapters

- **async**: In-process, good for development (single server)
- **redis**: Production-ready, supports multiple servers
- **postgresql**: Uses Postgres NOTIFY/LISTEN

## Mount the Server

```ruby
# config/routes.rb
Rails.application.routes.draw do
  mount ActionCable.server => '/cable'

  # Your other routes...
end
```

## The Connection Class

Handles WebSocket authentication:

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
      # Session-based auth
      if verified_user = User.find_by(id: cookies.encrypted[:user_id])
        verified_user
      else
        reject_unauthorized_connection
      end
    end
  end
end
```

### Alternative: Token Authentication

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
      token = request.params[:token]
      if verified_user = User.find_by(auth_token: token)
        verified_user
      else
        reject_unauthorized_connection
      end
    end
  end
end
```

Client connects with:
```javascript
createConsumer(`/cable?token=${userToken}`)
```

## The Channel Base Class

```ruby
# app/channels/application_cable/channel.rb
module ApplicationCable
  class Channel < ActionCable::Channel::Base
    # Shared channel logic
  end
end
```

## Client-Side Setup

Rails generates JavaScript for Action Cable:

```javascript
// app/javascript/channels/consumer.js
import { createConsumer } from "@rails/actioncable"

export default createConsumer()
```

## Import Maps Configuration

```ruby
# config/importmap.rb
pin "@rails/actioncable", to: "actioncable.esm.js"
pin_all_from "app/javascript/channels", under: "channels"
```

## Testing Connection

Start Rails and check the console:

```
Started GET "/cable" [WebSocket]
Successfully upgraded to WebSocket
```

In browser console:
```javascript
App.cable.connection.isOpen()  // true
```

## Resources

- [Action Cable Configuration](https://guides.rubyonrails.org/action_cable_overview.html#configuration) — Configuring Action Cable

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*