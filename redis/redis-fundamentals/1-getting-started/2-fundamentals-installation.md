---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-installation"
---

# Installing Redis

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

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*