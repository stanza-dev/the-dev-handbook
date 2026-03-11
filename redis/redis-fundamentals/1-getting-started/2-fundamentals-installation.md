---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-installation"
---

# Installing Redis

## Introduction

Before you can use Redis, you need to install the server and the CLI client. Redis runs natively on Linux and macOS, and Docker provides a consistent experience on any platform including Windows.

## Key Concepts

- **redis-server**: The Redis server process that stores data and processes commands.
- **redis-cli**: The command-line interface for connecting to and interacting with a Redis server.
- **Port 6379**: The default TCP port Redis listens on.
- **redis.conf**: The configuration file controlling Redis behavior (memory limits, persistence, networking).

## Real World Context

In development, you typically run Redis locally via Homebrew or Docker. In production, Redis is deployed as a managed service (AWS ElastiCache, Redis Cloud) or as a containerized service in Kubernetes. Understanding local installation helps you debug and test before deploying.

## Deep Dive

Let's get Redis running on your system. Redis runs natively on Linux and macOS. For Windows, we recommend using Docker or WSL2.

## macOS Installation (Homebrew)

The easiest way to install Redis on macOS:

```bash
# Install Redis
brew install redis

# Start Redis as a background service
brew services start redis

# Or start it manually
redis-server

# Verify installation
redis-cli ping
# Should return: PONG
```

## Linux Installation (Ubuntu/Debian)

```bash
# Update package lists
sudo apt update

# Install Redis
sudo apt install redis-server

# Start and enable the service
sudo systemctl start redis-server
sudo systemctl enable redis-server

# Verify it's running
redis-cli ping
# Should return: PONG
```

## Docker Installation (All Platforms)

Docker provides the most consistent experience across platforms:

```bash
# Pull and run Redis
docker run --name my-redis -p 6379:6379 -d redis

# Connect to it
docker exec -it my-redis redis-cli
```

## Connecting with redis-cli

The `redis-cli` is the command-line interface for Redis:

```bash
# Connect to local Redis (default port 6379)
redis-cli

# Connect to specific host and port
redis-cli -h localhost -p 6379

# Test the connection
127.0.0.1:6379> PING
PONG
```

## Your First Commands

Once connected, try these basic commands:

```redis
# Store a value
127.0.0.1:6379> SET greeting "Hello, Redis!"
OK

# Retrieve it
127.0.0.1:6379> GET greeting
"Hello, Redis!"

# Delete it
127.0.0.1:6379> DEL greeting
(integer) 1

# Check if it exists
127.0.0.1:6379> EXISTS greeting
(integer) 0
```

## Configuration Basics

Redis configuration file location:
- macOS (Homebrew): `/opt/homebrew/etc/redis.conf`
- Linux: `/etc/redis/redis.conf`

Key settings to know:

```
# Network
bind 127.0.0.1          # Only accept local connections
port 6379               # Default port

# Memory
maxmemory 100mb         # Memory limit
maxmemory-policy allkeys-lru  # Eviction policy

# Persistence
save 900 1              # Save after 900 seconds if 1 key changed
appendonly yes          # Enable AOF persistence
```

📖 [Redis Installation Guide](https://redis.io/docs/latest/operate/oss_and_stack/install/)

## Common Pitfalls

1. **Exposing Redis to the internet without authentication** — By default Redis binds to localhost with no password. Never expose port 6379 publicly without setting a password and firewall rules.
2. **Forgetting to start the service** — After installation, Redis doesn't always auto-start. Use `brew services start redis` or `systemctl enable redis-server` to ensure it starts on boot.

## Best Practices

1. **Use Docker for consistent environments** — Docker ensures the same Redis version across development, CI, and staging.
2. **Verify with PING** — After installation, always run `redis-cli ping` to confirm connectivity before proceeding.

## Summary

- Install Redis via Homebrew (macOS), apt (Linux), or Docker (any platform).
- The default port is 6379 and the CLI tool is redis-cli.
- Always verify installation with `redis-cli ping` (should return PONG).
- Configuration lives in redis.conf — key settings include bind, port, maxmemory, and persistence.

## Code Examples

**The two most common ways to install Redis — Homebrew for macOS and Docker for any platform**

```bash
# Install with Homebrew (macOS)
brew install redis
brew services start redis

# Or with Docker (any platform)
docker run --name my-redis -p 6379:6379 -d redis

# Verify connection
redis-cli ping
# Returns: PONG
```


## Resources

- [Redis Installation Guide](https://redis.io/docs/latest/operate/oss_and_stack/install/) — Official installation guide for all platforms

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*