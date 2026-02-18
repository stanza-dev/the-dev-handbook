---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-batch-processing"
---

# Batch Processing

Handle large data sets with batch processing patterns.

## Simple Batch Job

```ruby
class BulkEmailJob < ApplicationJob
  def perform(user_ids)
    User.where(id: user_ids).find_each do |user|
      UserMailer.newsletter(user).deliver_later
    end
  end
end

# Enqueue in batches
User.subscribed.pluck(:id).each_slice(100) do |batch|
  BulkEmailJob.perform_later(batch)
end
```

## Manual Batch Tracking

```ruby
class BatchProcessor
  def self.process(items, batch_size: 100)
    batch_id = SecureRandom.uuid
    total_batches = (items.count.to_f / batch_size).ceil
    
    Rails.cache.write("batch:#{batch_id}:total", total_batches)
    Rails.cache.write("batch:#{batch_id}:completed", 0)
    
    items.each_slice(batch_size).with_index do |batch, index|
      ProcessBatchJob.perform_later(
        batch_id: batch_id,
        batch_number: index,
        items: batch
      )
    end
    
    batch_id
  end
end

class ProcessBatchJob < ApplicationJob
  def perform(batch_id:, batch_number:, items:)
    items.each { |item| process_item(item) }
    
    completed = Rails.cache.increment("batch:#{batch_id}:completed")
    total = Rails.cache.read("batch:#{batch_id}:total")
    
    if completed >= total
      BatchCompletedJob.perform_later(batch_id)
    end
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*