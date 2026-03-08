---
source_course: "rails-security-hardening"
source_lesson: "rails-security-secrets-management"
---

# Secrets Management

## Introduction
API keys, database passwords, and encryption keys must never appear in version control. Rails provides encrypted credentials to manage secrets securely, ensuring they are available in production without exposing them in your codebase.

## Key Concepts
- **Rails Credentials**: An encrypted file (`credentials.yml.enc`) that stores secrets, decrypted at runtime using a master key.
- **Master Key**: A file (`config/master.key`) or environment variable (`RAILS_MASTER_KEY`) used to decrypt credentials.
- **Environment-Specific Credentials**: Separate encrypted credential files for different environments (development, staging, production).

## Real World Context
Committing API keys to Git is one of the most common security mistakes. Automated bots scan public repositories for exposed keys within seconds of a push. Rails credentials solve this by encrypting secrets in a file that is safe to commit.

## Deep Dive

### Rails Credentials

Rails encrypts credentials in `config/credentials.yml.enc`:

```bash
# Edit credentials (opens in editor)
RAILS_MASTER_KEY=xxx bin/rails credentials:edit

# Or with environment-specific credentials
bin/rails credentials:edit --environment production
```

The encrypted file is safe to commit to version control. Only the master key (which is gitignored) can decrypt it.

### Credentials Structure

Organize credentials by service:

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

Grouping by service makes credentials easy to find and update.

### Accessing Credentials

Read credentials in your application code:

```ruby
# Direct access
Rails.application.credentials.stripe[:secret_key]
Rails.application.credentials.aws[:access_key_id]

# With dig for safety (returns nil if any key is missing)
Rails.application.credentials.dig(:stripe, :secret_key)
```

The `dig` method is safer because it returns `nil` instead of raising `NoMethodError` if a key is missing.

### Credential Validation

Verify all required credentials are present on startup:

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

This initializer fails fast if credentials are missing, preventing the application from running in an insecure state.

## Common Pitfalls
1. **Committing master.key to Git** — The master key must never be in version control. Rails adds it to `.gitignore` by default, but verify this.
2. **Using ENV vars for everything** — While environment variables work, Rails credentials are more organized and auditable for application-specific secrets.

## Best Practices
1. **Use environment-specific credentials** — Separate files for development, staging, and production prevent accidental use of production keys in development.
2. **Validate credentials on startup** — Fail fast if required credentials are missing rather than discovering the problem at runtime.

## Summary
- Rails credentials encrypt secrets in a file safe for version control.
- The master key decrypts credentials and must never be committed to Git.
- Use `dig` for safe credential access that handles missing keys gracefully.
- Validate required credentials on application startup.
- Use environment-specific credential files for different deployment stages.

## Code Examples

**Safe credential access with dig and startup validation to fail fast on missing secrets**

```ruby
# Accessing encrypted credentials
Rails.application.credentials.dig(:stripe, :secret_key)

# Validating on startup
Rails.application.config.after_initialize do
  [:stripe, :secret_key].tap do |path|
    unless Rails.application.credentials.dig(*path)
      raise "Missing credential: #{path.join('.')}"
    end
  end
end

# Environment-specific credentials:
# bin/rails credentials:edit --environment production
```


## Resources

- [Rails Credentials Guide](https://guides.rubyonrails.org/security.html#custom-credentials) — Official Rails guide on managing encrypted credentials

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*