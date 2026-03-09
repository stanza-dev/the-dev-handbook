---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-backup-restore"
---

# Database Backups and Recovery

## Introduction
Regular backups are your safety net against data loss from hardware failures, human error, or security incidents. A backup strategy that isn't tested is no strategy at all.

## Key Concepts
- **pg_dump**: PostgreSQL's built-in tool for creating logical backups — exports data as SQL or a custom binary format.
- **Point-in-Time Recovery (PITR)**: Restoring to any moment in time using base backups plus Write-Ahead Log (WAL) archives.
- **Backup Verification**: Periodically restoring backups to a test database to ensure they're valid and complete.

## Real World Context
A developer accidentally runs `DELETE FROM orders` without a WHERE clause. Without tested backups, that data is gone forever. With daily backups and WAL archiving, you can restore to one minute before the mistake.

## Deep Dive

### Manual PostgreSQL Backup

```bash
# Custom format (compressed, supports parallel restore)
pg_dump -h localhost -U postgres -d myapp_production -F c -f backup.dump

# SQL format (human-readable)
pg_dump -h localhost -U postgres -d myapp_production | gzip > backup.sql.gz
```

### Automated Backup Job

```ruby
class DatabaseBackupJob < ApplicationJob
  queue_as :maintenance

  def perform
    timestamp = Time.current.strftime('%Y%m%d_%H%M%S')
    filename = "backup_#{timestamp}.dump"

    system("pg_dump -F c -f /tmp/#{filename} $DATABASE_URL")

    File.open("/tmp/#{filename}") do |file|
      S3_BUCKET.put_object(
        key: "backups/#{filename}",
        body: file,
        storage_class: 'STANDARD_IA'
      )
    end

    FileUtils.rm("/tmp/#{filename}")
  end
end
```

### Restore from Backup

```bash
pg_restore -h localhost -U postgres -d myapp_production -c backup.dump
```

### Backup via Kamal

```bash
# Execute pg_dump inside the database accessory
kamal accessory exec db 'pg_dump -U postgres myapp_production -F c' > backup.dump
```

## Common Pitfalls
1. **Never testing restores** — A backup you've never restored might be corrupt or incomplete. Test restores monthly.
2. **Storing backups on the same server** — If the server fails, you lose both the database and the backups. Always use off-site storage (S3, GCS).

## Best Practices
1. **Automate daily backups** — Schedule `DatabaseBackupJob` to run daily and upload to S3 with lifecycle policies.
2. **Retain at least 30 days** — Keep daily backups for a month, weekly backups for a year, to handle late-discovered issues.

## Summary
- Use pg_dump with custom format (-F c) for efficient, restorable backups.
- Automate backups with a background job uploading to S3.
- Test restores regularly — untested backups are unreliable.
- Store backups off-site and retain at least 30 days.

## Code Examples

**PostgreSQL backup and restore commands — use custom format (-F c) for efficient compressed backups**

```bash
# Create backup
pg_dump -F c -d myapp_production -f backup.dump

# Restore backup (drops and recreates objects)
pg_restore -d myapp_production -c backup.dump

# Backup via Kamal accessory
kamal accessory exec db 'pg_dump -U postgres myapp' > backup.dump
```


## Resources

- [PostgreSQL Backup and Restore](https://www.postgresql.org/docs/current/backup.html) — Official PostgreSQL documentation on backup strategies

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*