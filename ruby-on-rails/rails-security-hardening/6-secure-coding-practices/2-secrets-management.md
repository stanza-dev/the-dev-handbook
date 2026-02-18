---
source_course: "rails-security-hardening"
source_lesson: "rails-security-secrets-management"
---

# Secrets Management

Never commit secrets to version control. Rails provides secure credential management.

## Rails Credentials

Rails encrypts credentials in `config/credentials.yml.enc`:

```bash
# Edit credentials (opens in editor)
RAILS_MASTER_KEY=xxx bin/rails credentials:edit

# Or with environment-specific credentials
bin/rails credentials:edit --environment production
```

## Credentials Structure

```yaml
# config/credentials.yml.enc (decrypted)
aws:
  access_key_id: AKIAXXXXXXXX
  secret_access_key: wJalrXXXXXXXX

stripe:
  publishable_key: pk_test_xxx
  secret_key: sk_test_xxx
  webhook_secret: whsec_xxx

secret_key_base: abc123...
```

## Accessing Credentials

```ruby
# In your code
Rails.application.credentials.stripe[:secret_key]
Rails.application.credentials.aws[:access_key_id]

# With dig for safety
Rails.application.credentials.dig(:stripe, :secret_key)
```

## Environment Variables (Alternative)

```ruby
# For credentials that vary by deployment
ENV['DATABASE_URL']
ENV.fetch('API_KEY')  # Raises if missing
ENV.fetch('OPTIONAL_KEY', 'default')
```

## Keeping Master Key Safe

```bash
# .gitignore - already included by default
config/master.key
config/credentials/*.key

# Set in production via environment variable
RAILS_MASTER_KEY=your-key-here
```

## Credential Validation

```ruby
# config/initializers/credentials_check.rb
Rails.application.config.after_initialize do
  required_credentials = [
    [:stripe, :secret_key],
    [:aws, :access_key_id],
    [:aws, :secret_access_key]
  ]
  
  required_credentials.each do |path|
    unless Rails.application.credentials.dig(*path)
      raise "Missing required credential: #{path.join('.')}"
    end
  end
end
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*