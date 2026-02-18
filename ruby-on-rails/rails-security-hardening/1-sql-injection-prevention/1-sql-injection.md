---
source_course: "rails-security-hardening"
source_lesson: "rails-security-understanding-sql-injection"
---

# Understanding SQL Injection

SQL injection is one of the most dangerous web vulnerabilities. It occurs when user input is directly interpolated into SQL queries.

## How SQL Injection Works

Consider this vulnerable code:

```ruby
# DANGEROUS - Never do this!
User.find_by("login = '#{params[:name]}' AND password = '#{params[:password]}'")
```

An attacker could input:
- Username: `' OR '1'='1`
- Password: `' OR '1'='1`

This transforms the query into:
```sql
SELECT * FROM users WHERE login = '' OR '1'='1' AND password = '' OR '1'='1'
```

Since `'1'='1'` is always true, this returns the first user, bypassing authentication entirely.

## Attack Consequences

SQL injection can lead to:
- **Authentication bypass**: Login as any user
- **Data theft**: Extract sensitive information
- **Data destruction**: DELETE or DROP tables
- **Privilege escalation**: Modify user roles

## Another Example: Data Extraction

```ruby
# Vulnerable search
Project.where("name = '#{params[:name]}'")
```

Attacker input: `') UNION SELECT id,login,password,1,1,1 FROM users --`

This injects a UNION query that returns user credentials.

## Rails Protection

Rails provides safe query methods that automatically escape input. Never use string interpolation with user input in queries.

Learn more at [SQL Injection Security Guide](https://guides.rubyonrails.org/security.html#sql-injection).

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*