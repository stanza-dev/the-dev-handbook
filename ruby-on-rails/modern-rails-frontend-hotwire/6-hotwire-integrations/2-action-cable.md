---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-hotwire-action-cable"
---

# Hotwire Action Cable

While Turbo Streams has built-in broadcasting, custom channels offer more control.

## Custom Channel

```ruby
# app/channels/presence_channel.rb
class PresenceChannel < ApplicationCable::Channel
  def subscribed
    stream_from "presence_#{params[:room_id]}"
    
    # Announce arrival
    ActionCable.server.broadcast(
      "presence_#{params[:room_id]}",
      type: 'join',
      user: current_user.as_json(only: [:id, :name])
    )
  end
  
  def unsubscribed
    ActionCable.server.broadcast(
      "presence_#{params[:room_id]}",
      type: 'leave',
      user_id: current_user.id
    )
  end
end
```

## Stimulus + Action Cable

```javascript
// controllers/presence_controller.js
import { Controller } from '@hotwired/stimulus'
import { createConsumer } from '@rails/actioncable'

export default class extends Controller {
  static targets = ['users']
  static values = { roomId: Number }
  
  connect() {
    this.channel = createConsumer().subscriptions.create(
      { channel: 'PresenceChannel', room_id: this.roomIdValue },
      {
        received: data => this.handleMessage(data)
      }
    )
  }
  
  disconnect() {
    this.channel.unsubscribe()
  }
  
  handleMessage(data) {
    switch (data.type) {
      case 'join':
        this.addUser(data.user)
        break
      case 'leave':
        this.removeUser(data.user_id)
        break
    }
  }
  
  addUser(user) {
    const element = document.createElement('div')
    element.id = `user_${user.id}`
    element.textContent = user.name
    this.usersTarget.appendChild(element)
  }
  
  removeUser(userId) {
    document.getElementById(`user_${userId}`)?.remove()
  }
}
```

## Combining with Turbo Streams

```javascript
// Receive HTML via custom channel, insert with Turbo
import { Turbo } from '@hotwired/turbo-rails'

received(data) {
  if (data.turbo_stream) {
    Turbo.renderStreamMessage(data.turbo_stream)
  }
}
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*