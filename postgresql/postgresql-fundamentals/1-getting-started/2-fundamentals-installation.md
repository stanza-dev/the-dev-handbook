---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-installation"
---

# Installing PostgreSQL

Let's get PostgreSQL installed on your system. The installation includes the server, command-line client (psql), and GUI tools.

## macOS Installation

The easiest way is using Homebrew:

```bash
# Install PostgreSQL 18
brew install postgresql@18

# Start the service
brew services start postgresql@18

# Verify installation
psql --version
```

Alternatively, use [Postgres.app](https://postgresapp.com/) for a simple GUI-based installation.

## Linux (Ubuntu/Debian)

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

## Windows

1. Download the installer from [postgresql.org/download/windows](https://www.postgresql.org/download/windows/)
2. Run the installer and follow the wizard
3. Remember the password you set for the `postgres` user
4. Use pgAdmin (included) or psql from the command line

## Connecting for the First Time

After installation, connect using psql:

```bash
# Connect as the default postgres user
psql -U postgres

# You should see the PostgreSQL prompt:
psql (18.0)
Type "help" for help.

postgres=#
```

## Essential psql Commands

| Command | Description |
|---------|-------------|
| `\l` | List all databases |
| `\c dbname` | Connect to a database |
| `\dt` | List tables in current database |
| `\d tablename` | Describe a table |
| `\q` | Quit psql |
| `\?` | Help on psql commands |

📖 [PostgreSQL Installation Guide](https://www.postgresql.org/docs/18/installation.html)

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*