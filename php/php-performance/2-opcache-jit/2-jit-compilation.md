---
source_course: "php-performance"
source_lesson: "php-performance-jit-compilation"
---

# JIT Compilation in PHP 8

## Introduction
Just-In-Time compilation converts PHP bytecode to machine code at runtime for maximum performance.

## Key Concepts
- **JIT (Just-In-Time) Compilation**: Compiles PHP bytecode to native machine code at runtime for CPU-intensive operations.
- **Tracing JIT**: Observes code execution patterns and compiles hot paths — the recommended JIT mode.
- **CRTO Configuration**: 4-digit value (CPU optimization, Register allocation, Trigger mode, Optimization level) for fine-grained JIT control.
- **JIT Buffer**: Dedicated memory (`jit_buffer_size`) for storing generated machine code, separate from OPcache memory.

## Real World Context
PHP's JIT compiler (introduced in PHP 8.0) provides 1.5-3x speedups for CPU-intensive workloads like mathematical computation, image processing, and machine learning. It has minimal impact on typical I/O-bound web applications (database queries, API calls), which is why profiling is essential before enabling JIT.

## Deep Dive
### Intro

Just-In-Time compilation converts PHP bytecode to machine code at runtime for maximum performance.

### Jit vs opcache

```
OPcache:   Source → Bytecode → Execute on Zend VM
JIT:       Source → Bytecode → Machine Code → Execute on CPU
```

### Enable jit

```ini
; php.ini
opcache.enable=1  ; Required
opcache.jit=1255  ; Enable JIT
opcache.jit_buffer_size=128M
```

### Jit configuration values

The `opcache.jit` value has 4 digits: `CRTO`

```
C - CPU-specific optimization
  0 = no optimization
  1 = enable AVX

R - Register allocation
  0 = no register allocation
  1 = local linear scan
  2 = global linear scan

T - Trigger mode
  0 = compile on script load
  1 = compile on first execution
  2 = compile on first execution and profile
  3 = compile on first call
  4 = compile based on @jit doc comment
  5 = compile hot functions based on call count

O - Optimization level
  0 = no JIT
  1 = minimal JIT
  2 = selective JIT
  3 = optimized JIT
  4 = optimized and speculative JIT
  5 = maximum JIT
```

### Recommended settings

```ini
; Tracing JIT (best for most apps)
opcache.jit=tracing
; Equivalent to: opcache.jit=1254

; Function JIT (more conservative)
opcache.jit=function
; Equivalent to: opcache.jit=1205

; Disable JIT
opcache.jit=off
; Equivalent to: opcache.jit=0
```

### When jit helps

```php
<?php
// JIT helps: CPU-intensive calculations
function fibonacci(int $n): int {
    if ($n <= 1) return $n;
    return fibonacci($n - 1) + fibonacci($n - 2);
}

// JIT helps: Loops with calculations
function processArray(array $data): float {
    $sum = 0;
    foreach ($data as $value) {
        $sum += sqrt($value) * log($value);
    }
    return $sum;
}

// JIT has less impact: I/O bound operations
function fetchData(): array {
    return $pdo->query('SELECT * FROM users')->fetchAll();
}
```

### Monitoring jit

```php
<?php
function jitStatus(): array
{
    $status = opcache_get_status();
    
    return [
        'enabled' => $status['jit']['enabled'],
        'on' => $status['jit']['on'],
        'kind' => $status['jit']['kind'],
        'buffer_size' => $status['jit']['buffer_size'],
        'buffer_free' => $status['jit']['buffer_free'],
    ];
}
```

### Benchmark comparison

```php
<?php
// Without JIT: ~2.5 seconds
// With JIT: ~0.5 seconds (5x faster)
$start = microtime(true);
$result = fibonacci(35);
echo microtime(true) - $start;
```

## Common Pitfalls
1. **Expecting JIT to speed up I/O-bound code** — JIT optimizes CPU execution, not database queries or network calls. Most PHP web apps won't see significant improvement.
2. **Setting `jit_buffer_size` too small** — If the buffer fills, JIT stops compiling new code. Monitor with `opcache_get_status()` and increase if the buffer is full.

## Best Practices
1. **Use `opcache.jit=tracing` for most applications** — The tracing JIT observes actual execution patterns and optimizes the hottest code paths.
2. **Benchmark before and after enabling JIT** — Measure actual request latency, not synthetic benchmarks, to determine if JIT benefits your specific workload.

## Summary
- JIT compiles PHP bytecode to native machine code, benefiting CPU-intensive operations.
- Use `opcache.jit=tracing` (equivalent to 1254) for the recommended tracing mode.
- JIT has minimal impact on I/O-bound web applications — benchmark your specific workload.

## Code Examples

**JIT performance benchmark**

```php
<?php
declare(strict_types=1);

// JIT benchmark comparison
class JitBenchmark
{
    public function run(): array
    {
        return [
            'fibonacci' => $this->benchFibonacci(),
            'mandelbrot' => $this->benchMandelbrot(),
            'string_ops' => $this->benchStringOps(),
            'array_ops' => $this->benchArrayOps(),
        ];
    }
    
    private function benchFibonacci(): float
    {
        $start = microtime(true);
        for ($i = 0; $i < 30; $i++) {
            $this->fibonacci(25);
        }
        return (microtime(true) - $start) * 1000;
    }
    
    private function fibonacci(int $n): int
    {
        if ($n <= 1) return $n;
        return $this->fibonacci($n - 1) + $this->fibonacci($n - 2);
    }
    
    private function benchMandelbrot(): float
    {
        $start = microtime(true);
        $size = 200;
        
        for ($y = 0; $y < $size; $y++) {
            for ($x = 0; $x < $size; $x++) {
                $zr = $zi = 0;
                $cr = ($x / $size) * 3.5 - 2.5;
                $ci = ($y / $size) * 2 - 1;
                
                for ($i = 0; $i < 100; $i++) {
                    $temp = $zr * $zr - $zi * $zi + $cr;
                    $zi = 2 * $zr * $zi + $ci;
                    $zr = $temp;
                    if ($zr * $zr + $zi * $zi > 4) break;
                }
            }
        }
        
        return (microtime(true) - $start) * 1000;
    }
    
    private function benchStringOps(): float
    {
        $start = microtime(true);
        
        for ($i = 0; $i < 100000; $i++) {
            $str = 'Hello World ' . $i;
            $upper = strtoupper($str);
            $replaced = str_replace('WORLD', 'PHP', $upper);
            $len = strlen($replaced);
        }
        
        return (microtime(true) - $start) * 1000;
    }
    
    private function benchArrayOps(): float
    {
        $start = microtime(true);
        
        $data = range(1, 10000);
        
        for ($i = 0; $i < 100; $i++) {
            $filtered = array_filter($data, fn($n) => $n % 2 === 0);
            $mapped = array_map(fn($n) => $n * 2, $filtered);
            $sum = array_sum($mapped);
        }
        
        return (microtime(true) - $start) * 1000;
    }
}

// Run benchmark
$bench = new JitBenchmark();
$results = $bench->run();

echo "JIT Status: " . (opcache_get_status()['jit']['on'] ? 'ON' : 'OFF') . "\n";
foreach ($results as $name => $ms) {
    printf("%s: %.2f ms\n", $name, $ms);
}
?>
```


## Resources

- [PHP JIT](https://www.php.net/manual/en/opcache.configuration.php#ini.opcache.jit) — PHP JIT configuration

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*