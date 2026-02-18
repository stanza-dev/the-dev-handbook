---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-streams-patterns"
---

# Streams Patterns

Combine Turbo Streams with other features for powerful UX.

## Flash Messages via Stream

```ruby
class ApplicationController < ActionController::Base
  def stream_flash(type, message)
    turbo_stream.prepend 'flash_messages',
      partial: 'shared/flash',
      locals: { type: type, message: message }
  end
end

def create
  @product = Product.create!(product_params)
  
  respond_to do |format|
    format.turbo_stream {
      render turbo_stream: [
        turbo_stream.prepend('products', @product),
        stream_flash(:notice, 'Product created!')
      ]
    }
  end
end
```

## Optimistic UI

```javascript
// Show comment immediately, replace when server responds
controllers/optimistic_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  submit(event) {
    const form = event.target
    const content = form.querySelector('[name="comment[body]"]').value
    
    // Add temporary comment
    const temp = document.createElement('div')
    temp.id = 'temp_comment'
    temp.className = 'comment pending'
    temp.textContent = content
    
    document.getElementById('comments').prepend(temp)
  }
}
```

## Conditional Broadcasting

```ruby
class Message < ApplicationRecord
  after_create_commit :broadcast_message
  
  private
  
  def broadcast_message
    conversation.users.each do |user|
      broadcast_append_to [
        user, conversation
      ],
      target: 'messages',
      partial: 'messages/message',
      locals: {
        message: self,
        current_user: user  # Personalized rendering
      }
    end
  end
end
```

## Rate Limiting Broadcasts

```ruby
class TypingIndicator < ApplicationRecord
  after_save :broadcast_typing
  
  def broadcast_typing
    # Debounce to avoid too many broadcasts
    Rails.cache.fetch("typing_#{conversation_id}_#{user_id}", expires_in: 2.seconds) do
      broadcast_replace_to conversation, target: 'typing_indicator'
      true
    end
  end
end
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*