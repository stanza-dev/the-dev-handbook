---
source_course: "php-performance"
source_lesson: "php-performance-load-balancing"
---

# Load Balancing PHP Applications

Load balancing distributes incoming traffic across multiple PHP servers to improve performance and reliability.

## Load Balancing Strategies

### Round Robin
```nginx
# nginx.conf
upstream php_servers {
    server 192.168.1.1:9000;
    server 192.168.1.2:9000;
    server 192.168.1.3:9000;
}
```

### Weighted Distribution
```nginx
upstream php_servers {
    server 192.168.1.1:9000 weight=3;  # Gets 3x traffic
    server 192.168.1.2:9000 weight=2;
    server 192.168.1.3:9000 weight=1;
}
```

### Least Connections
```nginx
upstream php_servers {
    least_conn;
    server 192.168.1.1:9000;
    server 192.168.1.2:9000;
}
```

### IP Hash (Sticky Sessions)
```nginx
upstream php_servers {
    ip_hash;  # Same IP always goes to same server
    server 192.168.1.1:9000;
    server 192.168.1.2:9000;
}
```

## Health Checks

```php
<?php
// /health endpoint
header('Content-Type: application/json');

try {
    // Check database
    $pdo = new PDO($dsn);
    $pdo->query('SELECT 1');
    
    // Check cache
    $redis = new Redis();
    $redis->ping();
    
    // Check disk space
    $freeSpace = disk_free_space('/');
    if ($freeSpace < 1024 * 1024 * 100) {  // < 100MB
        throw new Exception('Low disk space');
    }
    
    echo json_encode([
        'status' => 'healthy',
        'timestamp' => time(),
    ]);
    
} catch (Throwable $e) {
    http_response_code(503);
    echo json_encode([
        'status' => 'unhealthy',
        'error' => $e->getMessage(),
    ]);
}
```

```nginx
upstream php_servers {
    server 192.168.1.1:9000;
    server 192.168.1.2:9000;
    
    # Health check
    health_check interval=5s passes=2 fails=3 uri=/health;
}
```

## Graceful Degradation

```php
<?php
class ResilientService
{
    public function getData(): array
    {
        try {
            return $this->fetchFromPrimary();
        } catch (Throwable $e) {
            // Log error
            error_log('Primary failed: ' . $e->getMessage());
            
            // Try fallback
            return $this->fetchFromCache();
        }
    }
    
    public function processRequest(): Response
    {
        // Circuit breaker pattern
        if ($this->circuitBreaker->isOpen('external-api')) {
            return $this->fallbackResponse();
        }
        
        try {
            $result = $this->callExternalApi();
            $this->circuitBreaker->recordSuccess('external-api');
            return $result;
            
        } catch (Throwable $e) {
            $this->circuitBreaker->recordFailure('external-api');
            return $this->fallbackResponse();
        }
    }
}
```

## Zero-Downtime Deployments

```bash
#!/bin/bash
# deploy.sh - Rolling deployment

SERVERS="server1 server2 server3"

for server in $SERVERS; do
    echo "Deploying to $server..."
    
    # Remove from load balancer
    ssh $server 'touch /var/www/maintenance.flag'
    sleep 5  # Wait for existing requests
    
    # Deploy new code
    ssh $server 'cd /var/www && git pull && composer install --no-dev'
    
    # Clear OPcache
    ssh $server 'php -r "opcache_reset();"'
    
    # Restart PHP-FPM
    ssh $server 'systemctl reload php-fpm'
    
    # Add back to load balancer
    ssh $server 'rm /var/www/maintenance.flag'
    
    echo "$server deployed successfully"
    sleep 10  # Wait before next server
done
```

## Resources

- [Nginx Load Balancing](https://nginx.org/en/docs/http/load_balancing.html) — Nginx load balancing documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*