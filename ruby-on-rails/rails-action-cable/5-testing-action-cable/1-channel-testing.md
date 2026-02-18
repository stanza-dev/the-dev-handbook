---
source_course: "rails-action-cable"
source_lesson: "rails-action-cable-channel-testing"
---

# Testing Channels

Rails provides built-in support for testing Action Cable channels.

## Test Setup

```ruby
# test/channels/application_cable/connection_test.rb
require "test_helper"

class ApplicationCable::ConnectionTest < ActionCable::Connection::TestCase
  test "connects with valid user" do
    user = users(:one)

    # Simulate cookies
    cookies.encrypted[:user_id] = user.id

    connect

    assert_equal user, connection.current_user
  end

  test "rejects connection without user" do
    assert_reject_connection { connect }
  end

  test "rejects connection with invalid user" do
    cookies.encrypted[:user_id] = 'invalid'

    assert_reject_connection { connect }
  end
end
```

## Channel Tests

```ruby
# test/channels/chat_channel_test.rb
require "test_helper"

class ChatChannelTest < ActionCable::Channel::TestCase
  setup do
    @room = rooms(:one)
    @user = users(:one)
  end

  test "subscribes to room" do
    stub_connection current_user: @user

    subscribe room_id: @room.id

    assert subscription.confirmed?
    assert_has_stream_for @room
  end

  test "rejects subscription without room" do
    stub_connection current_user: @user

    subscribe room_id: nil

    assert subscription.rejected?
  end

  test "rejects subscription for non-member" do
    non_member = users(:non_member)
    stub_connection current_user: non_member

    subscribe room_id: @room.id

    assert subscription.rejected?
  end

  test "speak broadcasts message" do
    stub_connection current_user: @user
    subscribe room_id: @room.id

    assert_broadcast_on(@room, ->(data) {
      data[:type] == "message" && data[:body].include?("Hello")
    }) do
      perform :speak, message: "Hello"
    end
  end
end
```

## Testing Broadcasts

```ruby
# test/models/comment_test.rb
require "test_helper"

class CommentTest < ActiveSupport::TestCase
  include ActionCable::TestHelper

  test "broadcasts to article on create" do
    article = articles(:one)

    assert_broadcast_on(article, type: "new_comment") do
      Comment.create!(article: article, user: users(:one), body: "Test")
    end
  end

  test "broadcasts specific data" do
    article = articles(:one)

    assert_broadcasts(ArticleChannel.broadcasting_for(article), 1) do
      Comment.create!(article: article, user: users(:one), body: "Test")
    end
  end
end
```

## Integration Tests

```ruby
# test/integration/chat_test.rb
require "test_helper"

class ChatIntegrationTest < ActionDispatch::IntegrationTest
  include ActionCable::TestHelper

  test "creates message and broadcasts" do
    room = rooms(:one)
    user = users(:one)

    sign_in user

    assert_broadcasts("chat_room_#{room.id}", 1) do
      post room_messages_path(room), params: {
        message: { body: "Hello!" }
      }
    end

    assert_response :redirect
  end
end
```

## System Tests with Action Cable

```ruby
# test/system/chat_test.rb
require "application_system_test_case"

class ChatSystemTest < ApplicationSystemTestCase
  test "receives messages in real time" do
    room = rooms(:one)
    user = users(:one)
    other_user = users(:two)

    # Open two browser windows
    using_session(:user_one) do
      sign_in user
      visit room_path(room)
    end

    using_session(:user_two) do
      sign_in other_user
      visit room_path(room)

      fill_in "message", with: "Hello from user two!"
      click_button "Send"
    end

    # Verify user one sees the message
    using_session(:user_one) do
      assert_text "Hello from user two!"
    end
  end
end
```

## Resources

- [Testing Action Cable](https://guides.rubyonrails.org/testing.html#testing-action-cable) — Guide to testing Action Cable

---

> 📘 *This lesson is part of the [Real-Time Rails with Action Cable](https://stanza.dev/courses/rails-action-cable) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*