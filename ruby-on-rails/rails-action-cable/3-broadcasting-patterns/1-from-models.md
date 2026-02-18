---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-broadcast-from-models"
---

# Broadcasting from Models

Broadcast real-time updates directly from your Active Record models using callbacks.

## Basic Model Broadcasting

```ruby
class Comment < ApplicationRecord
  belongs_to :article
  belongs_to :user

  after_create_commit :broadcast_new_comment
  after_update_commit :broadcast_update
  after_destroy_commit :broadcast_removal

  private

  def broadcast_new_comment
    # Broadcast to article channel
    ArticleChannel.broadcast_to(article, {
      type: "new_comment",
      html: render_comment,
      count: article.comments.count
    })
  end

  def broadcast_update
    ArticleChannel.broadcast_to(article, {
      type: "update_comment",
      id: id,
      html: render_comment
    })
  end

  def broadcast_removal
    ArticleChannel.broadcast_to(article, {
      type: "remove_comment",
      id: id,
      count: article.comments.count
    })
  end

  def render_comment
    ApplicationController.renderer.render(
      partial: "comments/comment",
      locals: { comment: self }
    )
  end
end
```

## Using after_commit Correctly

```ruby
class Message < ApplicationRecord
  # WRONG: after_save runs inside the transaction
  # If the transaction rolls back, broadcast already happened!
  after_save :broadcast_message  # Don't do this!

  # CORRECT: after_commit runs after transaction succeeds
  after_create_commit :broadcast_new_message

  # Even better: after_commit with on: option
  after_commit :broadcast_new_message, on: :create
  after_commit :broadcast_update, on: :update
  after_commit :broadcast_deletion, on: :destroy
end
```

## Broadcasting with Background Jobs

For complex broadcasts, use a job:

```ruby
class Comment < ApplicationRecord
  after_create_commit :enqueue_broadcast

  private

  def enqueue_broadcast
    BroadcastCommentJob.perform_later(self)
  end
end

class BroadcastCommentJob < ApplicationJob
  def perform(comment)
    # Notify article subscribers
    ArticleChannel.broadcast_to(comment.article, {
      type: "new_comment",
      html: render_comment(comment)
    })

    # Notify mentioned users
    comment.mentioned_users.each do |user|
      NotificationsChannel.broadcast_to(user, {
        type: "mention",
        message: "#{comment.user.name} mentioned you",
        url: Rails.application.routes.url_helpers.article_path(comment.article)
      })
    end

    # Update comment count for all article viewers
    ActionCable.server.broadcast(
      "article_stats_#{comment.article_id}",
      { comments_count: comment.article.comments.count }
    )
  end

  private

  def render_comment(comment)
    ApplicationController.renderer.render(
      partial: "comments/comment",
      locals: { comment: comment }
    )
  end
end
```

## Rendering in Callbacks

```ruby
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true

  private

  def render_partial(partial, locals = {})
    ApplicationController.renderer.render(
      partial: partial,
      locals: locals
    )
  end
end

class Message < ApplicationRecord
  after_create_commit :broadcast_message

  private

  def broadcast_message
    ChatChannel.broadcast_to(room, {
      html: render_partial("messages/message", message: self)
    })
  end
end
```

## Conditional Broadcasting

```ruby
class Article < ApplicationRecord
  after_update_commit :broadcast_update, if: :should_broadcast?

  private

  def should_broadcast?
    # Only broadcast if visible attributes changed
    saved_change_to_title? || saved_change_to_body?
  end

  def broadcast_update
    # Only to users with access
    viewers.each do |user|
      ArticleChannel.broadcast_to([self, user], {
        type: "content_updated",
        title: title,
        updated_at: updated_at.iso8601
      })
    end
  end
end
```

## Resources

- [Active Record Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html) — Using callbacks with broadcasting

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*