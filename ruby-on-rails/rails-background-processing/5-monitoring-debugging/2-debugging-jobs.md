---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-debugging-jobs"
---

# Debugging Jobs

Techniques for debugging job issues.

## Logging

```ruby
class DebuggableJob < ApplicationJob
  def perform(id)
    logger.info "Starting job for #{id}"
    
    record = Record.find(id)
    logger.debug "Found record: #{record.attributes}"
    
    result = process(record)
    logger.info "Completed with result: #{result}"
    
  rescue => e
    logger.error "Job failed: #{e.message}"
    logger.error e.backtrace.first(10).join("\n")
    raise
  end
end
```

## Inspecting Failed Jobs

```ruby
# In Rails console

# View retry queue
rs = Sidekiq::RetrySet.new
rs.each do |job|
  puts "#{job.klass}: #{job.error_message}"
  puts "  Args: #{job.args}"
  puts "  Retry count: #{job['retry_count']}"
end

# View dead jobs
ds = Sidekiq::DeadSet.new
ds.each do |job|
  puts "#{job.klass}: #{job.error_message}"
end

# Retry a specific job
job = rs.find { |j| j.jid == 'abc123' }
job.retry

# Retry all
rs.retry_all

# Clear dead jobs
ds.clear
```

## Testing Jobs

```ruby
# test/jobs/process_order_job_test.rb
class ProcessOrderJobTest < ActiveJob::TestCase
  test 'processes order successfully' do
    order = orders(:pending)
    
    ProcessOrderJob.perform_now(order.id)
    
    order.reload
    assert_equal 'processed', order.status
  end
  
  test 'enqueues follow-up job' do
    order = orders(:pending)
    
    assert_enqueued_with(job: SendConfirmationJob) do
      ProcessOrderJob.perform_now(order.id)
    end
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*