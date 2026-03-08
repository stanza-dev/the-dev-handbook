---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-batch-processing"
---

# Batch Processing

## Introduction
When you need to process thousands of records, you cannot load them all at once. Batch processing patterns split large datasets into manageable chunks.

## Key Concepts
- **Fan-out Pattern**: One coordinator splits work into many parallel jobs.
- **find_each / each_slice**: Active Record methods for chunked processing.

## Real World Context
Sending newsletters to 100,000 users, re-indexing search, or migrating data all require batching.

## Deep Dive

### Fan-Out Pattern

```ruby
class BulkEmailCoordinatorJob < ApplicationJob
  def perform(campaign_id)
    campaign = Campaign.find(campaign_id)
    campaign.subscribers.pluck(:id).each_slice(100) do |batch_ids|
      BulkEmailBatchJob.perform_later(campaign_id, batch_ids)
    end
  end
end

class BulkEmailBatchJob < ApplicationJob
  def perform(campaign_id, user_ids)
    campaign = Campaign.find(campaign_id)
    User.where(id: user_ids).find_each do |user|
      CampaignMailer.send_campaign(campaign, user).deliver_now
    end
  end
end
```

### Tracking Progress

```ruby
class BatchImportCoordinatorJob < ApplicationJob
  def perform(import_id)
    import = Import.find(import_id)
    rows = CSV.read(import.file.path, headers: true)
    import.update!(total_rows: rows.size, processed_rows: 0, status: :processing)

    rows.each_slice(50).with_index do |batch, index|
      BatchImportChunkJob.perform_later(import_id, batch.map(&:to_h), index)
    end
  end
end
```

## Common Pitfalls
1. **Loading all records into memory** — Use `pluck(:id).each_slice` instead.
2. **Batch size too large or small** — 50-500 is usually the sweet spot.

## Best Practices
1. **Use a coordinator job to split work** — Easy to monitor and retry.
2. **Track progress in the database** — Let users see how far along the operation is.

## Summary
- Fan-out patterns split large tasks into parallel batch jobs.
- `pluck(:id).each_slice(N)` avoids loading everything into memory.
- Track progress with database counters.
- Batch sizes of 50-500 balance throughput and reliability.

## Code Examples

**Fan-out pattern — pluck IDs and split into batches for parallel processing**

```ruby
# Fan-out: split 10,000 users into batches of 100
User.subscribed.pluck(:id).each_slice(100) do |batch_ids|
  SendNewsletterBatchJob.perform_later(newsletter.id, batch_ids)
end
```


## Resources

- [Active Record — Batches](https://guides.rubyonrails.org/active_record_querying.html#retrieving-multiple-objects-in-batches) — find_each and find_in_batches

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*