---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-hll-internals"
---

# HyperLogLog Internals and Registers

## Introduction

Understanding how HyperLogLog works internally helps you reason about its accuracy, debug unexpected results, and explain its behavior to your team. This lesson demystifies the algorithm — from hashing to registers to the final cardinality estimate.

## Key Concepts

- **Register**: One of 16,384 small counters (6 bits each) that store the maximum number of leading zeros observed for its bucket.
- **Leading zeros**: The number of consecutive zero bits at the start of a hash value. Observing many leading zeros implies a large number of distinct items.
- **Dense encoding**: The full 12 KB representation using all 16,384 registers at 6 bits each.
- **Sparse encoding**: A compressed representation Redis uses when most registers are zero, saving memory for small HLLs.
- **Harmonic mean**: The aggregation method HyperLogLog uses across registers to reduce variance compared to a simple average.

## Real World Context

When a product manager asks "why is the unique visitor count slightly different each time I refresh?" or "can I trust this number for my board presentation?", knowing the internals lets you give a precise answer: the algorithm uses 16,384 independent estimators and combines them with a harmonic mean, producing a standard error of exactly 0.81%. The number is not random — it is a well-characterized statistical estimate.

## Deep Dive

### The Core Algorithm

The HyperLogLog algorithm works in four steps:

**Step 1: Hash the input.** Every item added via PFADD is hashed to produce a uniformly distributed 64-bit value.

**Step 2: Split the hash into bucket index and value.** The first 14 bits select one of 16,384 registers (2^14 = 16,384). The remaining 50 bits are examined for leading zeros.

**Step 3: Update the register.** If the number of leading zeros (plus one) in the remaining bits exceeds the current register value, the register is updated to the new, higher value.

```
Hash of "Alice": 00101101 01110010 ...
                  ^^^^^^^^^^^^^^    ^^^^^^^^^^^^^^^^^^^^
                  14 bits = 2925    remaining bits
                  (register index)  count leading zeros = 2
                                    store max(register[2925], 2+1)
```

**Step 4: Estimate cardinality.** The harmonic mean of 2^(-register[i]) across all 16,384 registers gives the cardinality estimate, with a correction factor.

### The 16,384 Registers

Redis uses exactly 16,384 registers (m = 2^14). Each register is 6 bits wide, enough to store values 0 through 63. This means HyperLogLog can track leading zero counts up to 63, which corresponds to hash spaces of up to 2^63 distinct items — far more than any practical use case.

Total memory: 16,384 registers x 6 bits = 98,304 bits = 12,288 bytes (12 KB).

### Dense vs Sparse Encoding

Redis optimizes small HyperLogLogs with sparse encoding:

- **Sparse**: When most registers are zero or have the same value, Redis stores them as run-length encoded sequences. A fresh HLL or one with few items might use only a few hundred bytes.
- **Dense**: Once enough registers have distinct values, Redis converts to the full 12 KB dense format. This conversion is automatic and irreversible.

You can observe this transition:

```redis
# Fresh HLL uses sparse encoding
PFADD small_hll "item1"
MEMORY USAGE small_hll
# Returns: ~200 bytes (sparse)

# After many distinct items, switches to dense
# (exact threshold depends on the data)
MEMORY USAGE large_hll
# Returns: ~12,304 bytes (dense)
```

### Why 0.81% Error?

The standard error formula for HyperLogLog is:

```
Standard Error = 1.04 / sqrt(m)
```

With m = 16,384:

```
1.04 / sqrt(16384) = 1.04 / 128 = 0.008125 = 0.81%
```

This is a statistical property of the harmonic mean estimator across the registers. More registers would mean lower error but more memory.

### The Prefix "PF"

PFADD, PFCOUNT, and PFMERGE are named after Philippe Flajolet, the mathematician who co-invented the HyperLogLog algorithm.

## Common Pitfalls

1. **Thinking sparse-to-dense conversion means data loss** — When Redis converts from sparse to dense encoding, no information is lost. The dense encoding is simply less compressed. Accuracy remains identical.
2. **Assuming more registers means better accuracy for small sets** — For very small cardinalities (under ~100), HyperLogLog uses bias correction. The algorithm is less accurate for very small counts than for large ones. For tiny sets, a regular Redis Set might be more appropriate.
3. **Confusing register values with counts** — A register value of 5 does not mean 5 items hashed to that bucket. It means the longest run of leading zeros observed in that bucket's portion of the hash space was 4 (register stores leading_zeros + 1).

## Best Practices

1. **Do not worry about encoding transitions** — Redis handles sparse-to-dense conversion automatically. You do not need to pre-allocate or configure anything.
2. **Trust the math for large cardinalities** — For cardinalities above 1,000, the 0.81% standard error is very reliable. For cardinalities below 100, consider using exact Sets.
3. **Use DEBUG OBJECT to inspect encoding** — During development, you can check whether a HLL is in sparse or dense encoding to understand memory behavior.

## Summary

- HyperLogLog uses 16,384 six-bit registers that track the maximum leading zeros observed per hash bucket.
- The harmonic mean across all registers produces the cardinality estimate with 0.81% standard error.
- Redis uses sparse encoding for small HLLs (a few hundred bytes) and automatically converts to dense encoding (12 KB) as more distinct items are added.
- The standard error of 0.81% comes from the formula 1.04/sqrt(16384).
- The "PF" prefix honors Philippe Flajolet, co-inventor of the algorithm.

## Code Examples

**Observing sparse vs dense encoding and verifying HyperLogLog accuracy**

```python
import redis

r = redis.Redis()

# Demonstrate sparse vs dense encoding
r.delete('hll:sparse_demo', 'hll:dense_demo')

# Sparse: few items, low memory
r.pfadd('hll:sparse_demo', 'item1', 'item2', 'item3')
sparse_mem = r.execute_command('MEMORY', 'USAGE', 'hll:sparse_demo')
print(f'Sparse encoding memory: {sparse_mem} bytes')
# Typically ~200 bytes

# Dense: many distinct items, 12 KB
for i in range(10000):
    r.pfadd('hll:dense_demo', f'item:{i}')
dense_mem = r.execute_command('MEMORY', 'USAGE', 'hll:dense_demo')
print(f'Dense encoding memory: {dense_mem} bytes')
# ~12,304 bytes

# Cardinality estimate
count = r.pfcount('hll:dense_demo')
print(f'Estimated cardinality: {count}')
print(f'True cardinality: 10000')
print(f'Error: {abs(count - 10000) / 10000:.2%}')
```


## Resources

- [HyperLogLog](https://redis.io/docs/latest/develop/data-types/probabilistic/hyperloglogs/) — Redis HyperLogLog documentation including encoding details
- [HyperLogLog: the analysis of a near-optimal cardinality estimation algorithm](https://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf) — The original HyperLogLog paper by Flajolet et al.

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*