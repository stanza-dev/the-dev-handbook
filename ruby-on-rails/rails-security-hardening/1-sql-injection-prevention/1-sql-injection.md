---
source_course: "rails-security-hardening"
source_lesson: "rails-security-understanding-sql-injection"
---

# Understanding SQL Injection

## Introduction
SQL injection is one of the most dangerous and common web vulnerabilities, consistently ranking in the OWASP Top 10. It occurs when user input is directly interpolated into SQL queries, allowing attackers to manipulate your database in unintended ways.

## Key Concepts
- **SQL Injection**: An attack where malicious SQL code is inserted into application queries through user input.
- **String Interpolation in SQL**: Directly embedding user input into query strings without escaping, creating a vulnerability.
- **Parameterized Query**: A query where user input is passed as separate parameters rather than embedded in the SQL string.

## Real World Context
SQL injection has been responsible for some of the largest data breaches in history. A single vulnerable query can expose your entire database — user credentials, financial data, personal information. Every Rails developer must understand this attack vector to write safe code.

## Deep Dive

Consider this vulnerable code that authenticates a user:

```ruby
# DANGEROUS - Never do this!
User.find_by("login = '#{params[:name]}' AND password = '#{params[:password]}'")
```

This code directly interpolates user input into the SQL query string. An attacker could submit specially crafted input to bypass authentication entirely.

For example, if an attacker enters `' OR '1'='1` as both the username and password, the resulting SQL becomes:

```sql
SELECT * FROM users WHERE login = '' OR '1'='1' AND password = '' OR '1'='1'
```

Since `'1'='1'` is always true, this query returns the first user in the database, bypassing authentication completely.

Beyond authentication bypass, attackers can extract data using UNION-based injection. Consider a vulnerable search feature:

```ruby
# Vulnerable search
Project.where("name = '#{params[:name]}'")
```

An attacker could input `') UNION SELECT id,login,password,1,1,1 FROM users --` to inject a UNION query that returns user credentials alongside normal results. The `--` comments out the rest of the original query.

## Common Pitfalls
1. **Using string interpolation with user input** — Even experienced developers sometimes use `"#{params[:value]}"` in queries for convenience. Always use parameterized queries instead.
2. **Trusting indirect user input** — Data from cookies, HTTP headers, or even database records that originated from user input can all be attack vectors. Treat all external data as untrusted.

## Best Practices
1. **Use Rails query methods** — Hash conditions, placeholder syntax, and `find_by` with hashes are all safe by default. Make them your muscle memory.
2. **Enable Brakeman in CI** — The Brakeman static analysis tool detects SQL injection vulnerabilities automatically. Run it on every pull request.

## Summary
- SQL injection occurs when user input is interpolated directly into SQL query strings.
- Attackers can bypass authentication, extract sensitive data, or destroy database records.
- Rails provides built-in protection through parameterized queries and hash conditions.
- Never use string interpolation (`#{}`) inside SQL query strings with user-supplied data.

## Code Examples

**A vulnerable login query using string interpolation — an attacker can bypass authentication by injecting always-true conditions**

```ruby
# DANGEROUS — string interpolation lets attackers inject SQL
User.find_by("login = '#{params[:name]}' AND password = '#{params[:password]}'")

# Attacker input: name = "' OR '1'='1", password = "' OR '1'='1"
# Resulting SQL:
# SELECT * FROM users WHERE login = '' OR '1'='1' AND password = '' OR '1'='1'
# => Returns first user, bypassing authentication!
```


## Resources

- [Rails Security Guide — SQL Injection](https://guides.rubyonrails.org/security.html#sql-injection) — Official Rails guide covering SQL injection prevention techniques
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) — Comprehensive reference for SQL injection prevention across frameworks

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*