---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-backup-restore"
---

# Backup Restore

Regular backups are essential for data protection.

## PostgreSQL Backups

### Manual Backup

```bash
# Full backup
pg_dump -h localhost -U postgres -d myapp_production -F c -f backup.dump

# With compression
pg_dump -h localhost -U postgres -d myapp_production | gzip > backup.sql.gz
```

### Automated Backups

```ruby
# app/jobs/database_backup_job.rb
class DatabaseBackupJob < ApplicationJob
  queue_as :maintenance
  
  def perform
    timestamp = Time.current.strftime('%Y%m%d_%H%M%S')
    filename = "backup_#{timestamp}.dump"
    
    # Create backup
    system("pg_dump -F c -f /tmp/#{filename} $DATABASE_URL")
    
    # Upload to S3
    File.open("/tmp/#{filename}") do |file|
      S3_BUCKET.put_object(
        key: "backups/#{filename}",
        body: file,
        storage_class: 'STANDARD_IA'
      )
    end
    
    # Cleanup old backups
    cleanup_old_backups
    
    # Cleanup local file
    FileUtils.rm("/tmp/#{filename}")
  end
  
  private
  
  def cleanup_old_backups
    # Keep last 30 days
    cutoff = 30.days.ago
    
    S3_BUCKET.objects(prefix: 'backups/').each do |object|
      if object.last_modified < cutoff
        object.delete
      end
    end
  end
end
```

### Restore from Backup

```bash
# Download backup
aws s3 cp s3://mybucket/backups/backup_20240115.dump ./backup.dump

# Restore
pg_restore -h localhost -U postgres -d myapp_production -c backup.dump

# Or for SQL format
psql -h localhost -U postgres -d myapp_production < backup.sql
```

## Point-in-Time Recovery

For PostgreSQL with WAL archiving:

```bash
# Enable WAL archiving in postgresql.conf
archive_mode = on
archive_command = 'aws s3 cp %p s3://mybucket/wal/%f'

# Restore to specific point
restore_command = 'aws s3 cp s3://mybucket/wal/%f %p'
recovery_target_time = '2024-01-15 14:30:00'
```

## Testing Backups

```ruby
class BackupVerificationJob < ApplicationJob
  def perform
    # Download latest backup
    latest = S3_BUCKET.objects(prefix: 'backups/').max_by(&:last_modified)
    
    # Restore to test database
    system("pg_restore -d myapp_backup_test #{latest.key}")
    
    # Verify record counts
    test_db = ActiveRecord::Base.establish_connection(:backup_test)
    
    %w[users orders products].each do |table|
      count = test_db.connection.select_value("SELECT COUNT(*) FROM #{table}")
      Rails.logger.info "Backup verification: #{table} has #{count} records"
    end
  end
end
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*