---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-inline-asm"
---

# Inline Assembly: asm!

## Introduction
The `asm!` macro lets you embed assembly instructions directly in Rust code. Stabilized in Rust 1.59, it provides a safe syntax for specifying inputs, outputs, and clobbers. This is essential for accessing CPU-specific instructions not exposed through Rust or LLVM intrinsics.

## Key Concepts
- **`asm!`**: A macro for embedding inline assembly with typed operands.
- **Operand types**: `in`, `out`, `inout`, `lateout` specify how values flow between Rust and assembly.
- **Clobbers**: Registers that the assembly modifies but Rust doesn't use as outputs.
- **Options**: `nostack`, `nomem`, `pure` tell the compiler what side effects the assembly has.
- **`global_asm!`**: For defining entire assembly functions outside of Rust functions.

## Real World Context
Inline assembly is used in cryptographic libraries (AES-NI instructions), operating system kernels (system calls, interrupt handlers), and performance-critical code (SIMD without compiler support). The Linux kernel's Rust support uses `asm!` extensively.

## Deep Dive

### Basic Syntax

The `asm!` macro takes an assembly template string followed by operands:

```rust
use std::arch::asm;

fn add_asm(a: u64, b: u64) -> u64 {
    let result: u64;
    unsafe {
        asm!(
            "add {0}, {1}",
            inout(reg) a => result,
            in(reg) b,
        );
    }
    result
}
```

The `inout(reg) a => result` means: put `a` in a register, use it as input, and store the result of the `add` instruction in `result`.

### Operand Types

| Operand | Meaning | Example |
|---------|---------|--------|
| `in(reg)` | Input in a general register | `in(reg) value` |
| `out(reg)` | Output in a general register | `out(reg) result` |
| `inout(reg)` | Register used as both input and output | `inout(reg) a => b` |
| `lateout(reg)` | Output that may reuse an input register | `lateout(reg) result` |
| `const` | Compile-time constant | `const 42` |
| `sym` | Symbol reference (function/static) | `sym my_function` |

You can also specify exact registers:

```rust
unsafe {
    asm!(
        "cpuid",
        inout("eax") leaf => eax,
        out("ebx") ebx,
        out("ecx") ecx,
        out("edx") edx,
    );
}
```

Named registers like `"eax"` give you precise control.

### Options

Options tell the compiler what the assembly does:

```rust
unsafe {
    asm!(
        "rdtsc",
        out("eax") low,
        out("edx") high,
        options(nostack, nomem),  // Does not touch stack or memory
    );
}
```

| Option | Meaning |
|--------|---------|
| `nostack` | Assembly does not use the stack |
| `nomem` | Assembly does not read or write memory |
| `pure` | No side effects (implies nomem) |
| `noreturn` | Assembly never returns |
| `att_syntax` | Use AT&T syntax instead of Intel |

### CPUID Example

```rust
#[cfg(target_arch = "x86_64")]
fn cpuid(leaf: u32) -> (u32, u32, u32, u32) {
    let (eax, ebx, ecx, edx);
    unsafe {
        asm!(
            "cpuid",
            inout("eax") leaf => eax,
            out("ebx") ebx,
            out("ecx") ecx,
            out("edx") edx,
        );
    }
    (eax, ebx, ecx, edx)
}
```

This queries CPU capabilities, which is essential for runtime feature detection.

### `global_asm!` for Full Functions

For entire assembly functions, use `global_asm!`:

```rust
use std::arch::global_asm;

global_asm!(
    ".global fast_add",
    "fast_add:",
    "    mov rax, rdi",
    "    add rax, rsi",
    "    ret",
);

unsafe extern "C" {
    fn fast_add(a: u64, b: u64) -> u64;
}
```

`global_asm!` is placed at module scope, not inside a function. The function is declared via `unsafe extern "C"` (Edition 2024).

## Common Pitfalls
1. **Forgetting clobbers** — If your assembly modifies a register that's not listed as an output or clobber, the compiler may use that register for something else, causing silent corruption.
2. **Wrong options** — Claiming `nomem` when the assembly reads memory allows the compiler to reorder memory accesses incorrectly.
3. **Architecture-specific code without `cfg`** — Always gate assembly behind `#[cfg(target_arch = "...")]` or it will fail on other platforms.

## Best Practices
1. **Use the highest-level API available** — Prefer `std::arch::x86_64` intrinsics over inline assembly when possible.
2. **Always specify `options`** — Telling the compiler what the assembly does (or doesn't do) enables better optimization.
3. **Test on all target architectures** — Assembly is not portable. Use `cfg` to provide fallback implementations.

## Summary
- `asm!` (stable since 1.59) embeds inline assembly with typed operands.
- Operands specify how values flow: `in`, `out`, `inout`, `lateout`.
- Options like `nostack` and `nomem` help the compiler optimize around assembly.
- `global_asm!` defines entire assembly functions at module scope.
- Always gate architecture-specific assembly behind `#[cfg(target_arch)]`.

## Code Examples

**x86-64 inline assembly examples — timestamp counter (rdtsc), pause for spin loops, and memory fence, all using proper options**

```rust
#[cfg(target_arch = "x86_64")]
mod x86 {
    use std::arch::asm;

    /// Read the CPU timestamp counter (cycle count)
    pub fn rdtsc() -> u64 {
        let low: u32;
        let high: u32;
        unsafe {
            asm!(
                "rdtsc",
                out("eax") low,
                out("edx") high,
                options(nostack, nomem),
            );
        }
        ((high as u64) << 32) | (low as u64)
    }

    /// Pause instruction for spin loops (reduces power consumption)
    #[inline(always)]
    pub fn pause() {
        unsafe {
            asm!("pause", options(nostack, nomem));
        }
    }

    /// Memory fence (full barrier)
    pub fn mfence() {
        unsafe {
            asm!("mfence", options(nostack));
        }
    }
}

#[cfg(target_arch = "x86_64")]
fn main() {
    let start = x86::rdtsc();
    // ... work ...
    let end = x86::rdtsc();
    println!("Elapsed cycles: {}", end - start);
}
```


## Resources

- [Inline Assembly — Rust Reference](https://doc.rust-lang.org/reference/inline-assembly.html) — Official specification of the asm! macro syntax and semantics

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*