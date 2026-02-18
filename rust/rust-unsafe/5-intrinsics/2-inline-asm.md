---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-inline-asm"
---

# asm! Macro (Stable Since 1.59)

```rust
use std::arch::asm;

fn add(a: u64, b: u64) -> u64 {
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

## Operand Types

| Operand | Meaning |
|---------|----------|
| `in(reg)` | Input in a register |
| `out(reg)` | Output register |
| `inout(reg)` | Input and output |
| `lateout(reg)` | Output, can reuse input reg |
| `const` | Compile-time constant |
| `sym` | Symbol reference |

## Example: CPUID

```rust
use std::arch::asm;

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

## Clobbers & Options

```rust
unsafe {
    asm!(
        "syscall",
        in("rax") syscall_number,
        // These registers are modified
        out("rcx") _,
        out("r11") _,
        // Options
        options(nostack, nomem),
    );
}
```

## global_asm! for Full Functions

```rust
use std::arch::global_asm;

global_asm!(
    ".global my_asm_func",
    "my_asm_func:",
    "    mov rax, rdi",
    "    add rax, rsi",
    "    ret",
);

extern "C" {
    fn my_asm_func(a: u64, b: u64) -> u64;
}
```

See [Inline Assembly](https://doc.rust-lang.org/reference/inline-assembly.html).

## Code Examples

**x86 timestamp and sync**

```rust
#[cfg(target_arch = "x86_64")]
mod x86 {
    use std::arch::asm;
    
    /// Read the CPU timestamp counter
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
    
    /// Pause instruction for spin loops
    #[inline(always)]
    pub fn pause() {
        unsafe {
            asm!("pause", options(nostack, nomem));
        }
    }
    
    /// Memory fence
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
    println!("Cycles: {}", end - start);
}
```


## Resources

- [Inline Assembly](https://doc.rust-lang.org/reference/inline-assembly.html) — Official inline assembly reference

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*