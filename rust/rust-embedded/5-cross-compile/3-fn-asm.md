---
source_course: "rust-embedded"
source_lesson: "rust-embedded-naked-fn-asm"
---

# Naked Functions and Inline Assembly

## Introduction
Sometimes you need total control over generated assembly — custom interrupt trampolines, context switches in an RTOS, or bootloader jump sequences. Rust's `naked_asm!` macro and `#[unsafe(naked)]` attribute (stabilized in Rust 1.88) give you this control while integrating cleanly with the Rust build system.

## Key Concepts
- **`#[unsafe(naked)]`**: An attribute that prevents the compiler from generating a function prologue/epilogue (no stack frame setup, no register saves).
- **`naked_asm!`**: A macro for writing assembly in naked functions — the only thing allowed in a naked function body.
- **Inline assembly (`asm!`)**: The standard `core::arch::asm!` macro for embedding assembly instructions in regular Rust functions.

## Real World Context
RTOS context switches must save and restore all CPU registers in a specific order. Bootloader jump sequences must set the stack pointer and branch to an address without any compiler-inserted code. These operations are impossible with regular Rust functions because the compiler adds prologue/epilogue code.

## Deep Dive

### Naked Functions

A naked function has no compiler-generated prologue or epilogue — you control every instruction:

```rust
use core::arch::naked_asm;

#[unsafe(naked)]
pub unsafe extern "C" fn context_switch(old_sp: *mut usize, new_sp: usize) {
    naked_asm!(
        "push {{r4-r11, lr}}",   // Save callee-saved registers
        "str sp, [r0]",          // Save current stack pointer
        "mov sp, r1",            // Load new stack pointer
        "pop {{r4-r11, lr}}",    // Restore registers
        "bx lr",                 // Return to new context
    );
}
```

The compiler emits exactly the instructions you wrote — nothing more. This is essential for context switches where the prologue would corrupt the register state you are trying to save.

### Bootloader Jump

Jumping to application code from a bootloader requires setting the stack pointer and branching:

```rust
use core::arch::naked_asm;

#[unsafe(naked)]
pub unsafe extern "C" fn jump_to_app(app_vector_table: u32) {
    naked_asm!(
        "ldr r1, [r0]      ",  // Load initial stack pointer from vector table
        "msr msp, r1       ",  // Set Main Stack Pointer
        "ldr r1, [r0, #4]  ",  // Load reset vector (entry point)
        "bx  r1            ",  // Branch to application
    );
}
```

This function reads the application's vector table to get the stack pointer and entry point, then jumps there. No Rust prologue is acceptable here because we are completely replacing the execution context.

### Inline Assembly in Regular Functions

For smaller assembly snippets, use `asm!` in regular functions:

```rust
use core::arch::asm;

fn delay_cycles(cycles: u32) {
    unsafe {
        asm!(
            "1: subs {0}, {0}, #1",  // Subtract 1
            "   bne 1b",              // Loop if not zero
            inout(reg) cycles => _,
            options(nomem, nostack),
        );
    }
}
```

The `asm!` macro is used inside a regular function with a normal prologue. The `options(nomem, nostack)` tells the compiler this assembly does not access memory or the stack, enabling better optimization.

## Common Pitfalls
1. **Putting Rust code in a naked function** — Naked functions can ONLY contain a single `naked_asm!()` invocation. No `let` bindings, no function calls, no Rust expressions.
2. **Forgetting to return from naked functions** — The compiler does not insert a `ret` or `bx lr`. If you forget, execution falls through to whatever code follows.
3. **Wrong register conventions** — Naked functions must follow the platform's calling convention manually. On ARM, arguments are in r0-r3 and the return address is in lr.

## Best Practices
1. **Prefer `asm!` over `#[unsafe(naked)]`** — Use naked functions only when you need full control over the prologue/epilogue. For most cases, `asm!` in a regular function is safer and sufficient.
2. **Document the calling convention** — Add comments explaining which registers hold which arguments and what the function expects.
3. **Test with objdump** — Verify the generated assembly matches your expectations: `cargo objdump --release -- -d | grep my_function`.

## Summary
- `#[unsafe(naked)]` + `naked_asm!` gives complete control over generated assembly (stabilized in Rust 1.88).
- Use naked functions for context switches, bootloader jumps, and interrupt trampolines.
- Use `asm!` in regular functions for smaller assembly snippets.
- Naked functions can only contain `naked_asm!()` — no Rust code allowed.
- Always follow the platform calling convention and document register usage.

## Code Examples

**Bootloader jump function using #[unsafe(naked)] — sets the stack pointer from the application vector table and branches to the reset handler**

```rust
use core::arch::naked_asm;

/// Jump from bootloader to application firmware.
/// The app_base address points to the application's vector table.
///
/// # Safety
/// The application vector table must be valid and the application
/// code must be properly linked at app_base.
#[unsafe(naked)]
pub unsafe extern "C" fn jump_to_application(app_base: u32) {
    naked_asm!(
        // r0 = app_base (application vector table address)
        "ldr r1, [r0]      ",  // r1 = initial stack pointer
        "msr msp, r1       ",  // Set Main Stack Pointer
        "ldr r1, [r0, #4]  ",  // r1 = reset handler address
        "bx  r1            ",  // Branch to application entry point
    );
}

// Usage in bootloader:
// unsafe { jump_to_application(0x0800_8000); }
```


## Resources

- [Rust inline assembly reference](https://doc.rust-lang.org/reference/inline-assembly.html) — Official reference for asm! and naked_asm! macros

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*