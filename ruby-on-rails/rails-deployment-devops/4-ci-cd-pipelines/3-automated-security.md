---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-automated-security"
---

# Automated Security Scanning

## Introduction
Automated security scans catch vulnerabilities in your dependencies and code before they reach production. Rails has excellent tooling for both dependency auditing and static code analysis.

## Key Concepts
- **bundler-audit**: Scans your Gemfile.lock against a database of known vulnerable gem versions.
- **Brakeman**: A static analysis tool that scans Rails code for security vulnerabilities like SQL injection, XSS, and mass assignment.
- **Dependabot**: GitHub's automated dependency update tool that creates PRs when vulnerabilities are discovered.

## Real World Context
A Rails app using an outdated version of `nokogiri` with a known XML parsing vulnerability could be exploited to read server files. `bundler-audit` catches this and alerts you before deployment.

## Deep Dive

### bundler-audit

```ruby
# Gemfile
gem 'bundler-audit', require: false, group: :development
```

```bash
# Check for vulnerable gems
bundle exec bundler-audit check --update

# Output:
# Name: actionpack
# Version: 7.0.4
# CVE: CVE-2023-22796
# Criticality: High
# Solution: upgrade to >= 7.0.4.1
```

### Brakeman

```ruby
# Gemfile
gem 'brakeman', require: false, group: :development
```

```bash
# Scan for code vulnerabilities
bundle exec brakeman -q

# Output:
# +SECURITY WARNINGS+
# SQL Injection in UsersController#index
#   User.where("name = '#{params[:name]}'")
```

### CI Integration

```yaml
# .github/workflows/ci.yml
security:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: '3.4'
        bundler-cache: true
    - run: bundle exec bundler-audit check --update
    - run: bundle exec brakeman -q --no-pager
```

### Dependabot Configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: bundler
    directory: '/'
    schedule:
      interval: weekly
    open-pull-requests-limit: 10
```

## Common Pitfalls
1. **Ignoring Brakeman warnings** — False positives exist, but don't dismiss warnings without investigation. Use `brakeman -I` to create an ignore file for confirmed false positives.
2. **Not running security scans in CI** — Manual scans are forgotten. Automate them as required CI checks.

## Best Practices
1. **Block merges on security failures** — Make bundler-audit and Brakeman required status checks on your main branch.
2. **Enable Dependabot** — It creates PRs automatically when vulnerable dependencies are discovered.

## Summary
- bundler-audit checks gems against known vulnerability databases.
- Brakeman scans Rails code for SQL injection, XSS, and other issues.
- Run both in CI as required checks before merging.
- Enable Dependabot for automatic dependency update PRs.

## Code Examples

**Running bundler-audit and Brakeman — both should pass before deploying to production**

```bash
# Check for vulnerable gem versions
$ bundle exec bundler-audit check --update
No vulnerabilities found

# Scan code for security issues
$ bundle exec brakeman -q
+SUMMARY+
| Checks   | 136 |
| Warnings | 0   |

# Both pass — safe to deploy!
```


## Resources

- [Securing Rails Applications](https://guides.rubyonrails.org/security.html) — Official Rails guide on security best practices

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*