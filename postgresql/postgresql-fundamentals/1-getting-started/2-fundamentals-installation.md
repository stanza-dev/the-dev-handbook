---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-installation"
---

# Installing PostgreSQL

## Introduction
Before you can write queries, you need PostgreSQL running on your system. This lesson walks you through installation on macOS, Linux, and Windows, and shows you how to connect with psql — the command-line client — for the first time.

## Key Concepts
- **psql**: The interactive terminal for PostgreSQL. It lets you type SQL commands and see results immediately.
- **postgres user**: The default superuser account created during installation. You connect as this user when setting up the database.
- **Backslash commands**: Special psql commands (like `\l`, `\dt`) that start with a backslash and provide metadata about your database.

## Real World Context
Every developer who works with PostgreSQL needs to know how to install it and navigate psql. Whether you are setting up a local development environment or debugging a production issue via SSH, psql is your primary interface to the database.

## Deep Dive

### macOS Installation

The easiest way to install on macOS is using Homebrew. Run these commands in your terminal:

```bash
# Install PostgreSQL 18
brew install postgresql@18

# Start the service
brew services start postgresql@18

# Verify installation
psql --version
```

This installs the server, psql, and related utilities. Alternatively, you can use [Postgres.app](https://postgresapp.com/) for a simpler GUI-based installation.

### Linux (Ubuntu/Debian)

On Ubuntu or Debian, add the official PostgreSQL repository and install:

```bash
# Add PostgreSQL repository
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import repository signing key
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update and install
sudo apt-get update
sudo apt-get install postgresql-18

# Check service status
sudo systemctl status postgresql
```

The server starts automatically after installation on Linux.

### Windows

1. Download the installer from [postgresql.org/download/windows](https://www.postgresql.org/download/windows/)
2. Run the installer and follow the wizard
3. Remember the password you set for the `postgres` user
4. Use pgAdmin (included) or psql from the command line

### Connecting for the First Time

After installation, connect using psql to verify everything is working:

```bash
# Connect as the default postgres user
psql -U postgres

# You should see the PostgreSQL prompt:
psql (18.3)
Type "help" for help.

postgres=#
```

The `postgres=#` prompt means you are connected and ready to run SQL commands.

### Essential psql Commands

psql has built-in commands that start with a backslash. These are not SQL — they are psql-specific shortcuts:

| Command | Description |
|---------|-------------|
| `\l` | List all databases |
| `\c dbname` | Connect to a database |
| `\dt` | List tables in current database |
| `\d tablename` | Describe a table |
| `\q` | Quit psql |
| `\?` | Help on psql commands |

These commands save time when exploring a database. Use `\?` to see the full list of available commands.

## Common Pitfalls
1. **Forgetting the postgres password on Windows** — The installer asks you to set a password for the `postgres` user. If you forget it, you will need to edit `pg_hba.conf` to regain access.
2. **Port conflicts** — PostgreSQL defaults to port 5432. If another service uses that port, you will get a connection error. Check with `lsof -i :5432` on macOS/Linux.
3. **Not starting the service** — On macOS with Homebrew, you must explicitly start the service with `brew services start postgresql@18`.

## Best Practices
1. **Use a dedicated database per project** — Avoid putting everything in the default `postgres` database. Create a new database for each application.
2. **Learn the key psql shortcuts** — Knowing `\dt`, `\d`, and `\l` speeds up your daily workflow significantly.

## Summary
- Install PostgreSQL via Homebrew (macOS), apt (Linux), or the official installer (Windows).
- Connect with `psql -U postgres` and verify the prompt shows `postgres=#`.
- Use backslash commands (`\l`, `\dt`, `\d`, `\q`) to navigate databases and tables.
- PostgreSQL defaults to port 5432 and creates a `postgres` superuser account.

## Code Examples

**Common psql connection patterns — connect, list databases, and run SQL files from the command line**

```bash
# Connect to PostgreSQL and list all databases
psql -U postgres -c '\l'

# Connect to a specific database
psql -U postgres -d my_database

# Run a SQL file against a database
psql -U postgres -d my_database -f setup.sql
```


## Resources

- [PostgreSQL Installation Guide](https://www.postgresql.org/docs/18/installation.html) — Official installation instructions for all platforms

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*