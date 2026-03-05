---
source_course: "python-performance"
source_lesson: "python-performance-statistical-profiling"
---

# Statistical & Remote Profiling

## Introduction

Deterministic profilers add significant overhead by instrumenting every function call. Statistical profilers take a fundamentally different approach: they periodically sample the call stack, providing a low-overhead approximation of where time is spent. Python 3.14 also introduces groundbreaking remote debugging capabilities through PEP 768.

## Key Concepts

- **Statistical sampling**: Instead of tracing every call, the profiler reads the call stack at fixed intervals (e.g., every 1ms). Functions that appear in more samples are consuming more CPU time.
- **py-spy**: A sampling profiler written in Rust that attaches to running Python processes from outside, requiring no code changes or restarts.
- **scalene**: A hybrid CPU/memory/GPU profiler that combines statistical sampling with memory allocation tracking and separates Python vs native (C) time.
- **PEP 768 (Python 3.14)**: External Debugger Injection — allows attaching a debugger or running arbitrary Python code in a live process via `sys.remote_exec()` or `python -m pdb -p PID`.

## Real World Context

In production, you cannot restart a web server to add profiling decorators. py-spy can attach to a running process and generate a flame graph without any code changes or downtime. scalene helps identify whether slowness comes from Python code, C extensions, or memory allocation pressure. Python 3.14's PEP 768 lets you attach a debugger to a hung production process to inspect its state.

## Deep Dive

### py-spy: Zero-Overhead Profiling

py-spy reads the Python interpreter's internal data structures from another process, so it adds essentially zero overhead to the profiled program:

```bash
# Install
pip install py-spy

# Profile a running process by PID
py-spy top --pid 12345

# Record a flame graph
py-spy record --pid 12345 -o profile.svg

# Profile a script from start
py-spy record -o profile.svg -- python my_script.py
```

The `top` subcommand gives a live, htop-like view showing which functions are currently consuming CPU. The `record` subcommand produces an interactive SVG flame graph.

### scalene: CPU + Memory + GPU Profiling

scalene distinguishes between Python time and native (C library) time, and also tracks memory allocations:

```bash
pip install scalene

# Profile a script
scalene my_script.py

# Profile with GPU tracking
scalene --gpu my_script.py
```

scalene output shows per-line breakdowns with separate columns for Python time, native time, and memory allocations. This is invaluable when you suspect a C extension (like NumPy or pandas) is the bottleneck rather than your Python code.

### Profiling Async Code

Asynchronous code requires special handling because `await` points yield control, making wall-clock time misleading:

```python
import asyncio
import time

async def fetch_data(url: str) -> bytes:
    # Wall time is high (network I/O), but CPU time is low
    await asyncio.sleep(1.0)  # Simulated network delay
    return b"data"

async def process_batch(urls: list[str]):
    tasks = [fetch_data(url) for url in urls]
    return await asyncio.gather(*tasks)
```

For async code, py-spy's `--subprocesses` flag and scalene both handle coroutines well. cProfile does not understand `await` boundaries and will lump I/O wait time into function timing.

### Python 3.14: PEP 768 Remote Debugger

Python 3.14 introduces `sys.remote_exec()` and remote pdb attachment — a game-changer for production debugging:

```python
import sys

# From another Python process, inject code into a running process
sys.remote_exec(target_pid, "/path/to/debug_script.py")
```

The debug script runs inside the target process's interpreter:

```python
# debug_script.py — executed inside the target process
import traceback
import sys

# Dump all thread stacks to a file
with open("/tmp/thread_dump.txt", "w") as f:
    for thread_id, frame in sys._current_frames().items():
        f.write(f"\nThread {thread_id}:\n")
        traceback.print_stack(frame, file=f)
```

You can also attach pdb directly from the command line:

```bash
# Attach an interactive debugger to a running Python process
python -m pdb -p 12345
```

This is equivalent to GDB-style attach for Python. The target process pauses at the next safe point, and you get a full pdb prompt with access to all frames, variables, and the ability to evaluate expressions.

Key requirements for PEP 768:
- The target process must be Python 3.14+
- The injecting process must have OS-level permission to attach (same user or root)
- `sys.remote_exec()` runs the script in the main thread at the next opportunity (similar to signal handling)

## Common Pitfalls

- **Using cProfile for async code**: cProfile measures wall time per function, not CPU time. An `await asyncio.sleep(1)` will show 1 second of "function time" even though zero CPU was used. Use statistical profilers instead.
- **Forgetting that sampling profilers are probabilistic**: If a function runs for a very short time, it may not appear in any samples. Increase the sampling rate (`py-spy --rate 1000`) for finer granularity.
- **Running py-spy without sufficient permissions**: On Linux, py-spy needs ptrace access. Run with `sudo` or set `kernel.yama.ptrace_scope=0` for development.

## Best Practices

- Use py-spy for production profiling — it adds no overhead to the target process and requires no code changes.
- Use scalene when you need to distinguish Python time from C extension time, or when you suspect memory allocation pressure.
- In Python 3.14+, use `python -m pdb -p PID` to debug hung or slow production processes without restarts.

## Summary

- Statistical profilers (py-spy, scalene) sample the call stack periodically, adding near-zero overhead compared to deterministic profilers.
- py-spy attaches to running processes and produces flame graphs without code changes.
- scalene separates Python, native (C), and GPU time while also tracking memory allocations.
- Python 3.14's PEP 768 introduces `sys.remote_exec()` and `pdb -p PID` for injecting code and debuggers into live processes.
- For async code, prefer statistical profilers over cProfile, which conflates I/O wait time with CPU time.

## Code Examples

**Demonstrating Python 3.14's PEP 768 sys.remote_exec() to inject diagnostic code into a running process**

```python
# Example: Using sys.remote_exec() in Python 3.14 to inspect a live process
# --- Target process (long_running_server.py) ---
import time
import os

print(f"Server PID: {os.getpid()}")
while True:
    time.sleep(0.1)  # Simulate server loop

# --- From another terminal (Python 3.14+) ---
# python -c "import sys; sys.remote_exec(TARGET_PID, 'inspect_script.py')"

# --- inspect_script.py (runs inside target process) ---
import sys
import traceback

frames = sys._current_frames()
for tid, frame in frames.items():
    print(f"\nThread {tid}:")
    traceback.print_stack(frame)
```

**Common py-spy command patterns for attaching to processes and generating flame graphs**

```python
# py-spy command examples (run from the terminal, not Python)
#
# Attach to a running process and show live top-like view:
#   py-spy top --pid 12345
#
# Record a flame graph SVG from a running process:
#   py-spy record --pid 12345 -o flamegraph.svg
#
# Profile a script from start to finish:
#   py-spy record -o profile.svg -- python my_script.py
#
# Increase sampling rate for short-lived functions:
#   py-spy record --rate 1000 --pid 12345 -o detailed.svg
#
# Include native (C extension) frames:
#   py-spy record --native --pid 12345 -o native.svg

# In Python, you can launch py-spy programmatically:
import subprocess
import os

def profile_self(duration_seconds=10, output="self_profile.svg"):
    """Launch py-spy to profile the current process for N seconds."""
    pid = os.getpid()
    subprocess.Popen([
        "py-spy", "record",
        "--pid", str(pid),
        "--duration", str(duration_seconds),
        "-o", output
    ])
    print(f"py-spy recording PID {pid} for {duration_seconds}s -> {output}")
```


## Resources

- [PEP 768 — Safe external debugger interface for CPython](https://docs.python.org/3.14/whatsnew/3.14.html) — Python 3.14 What's New page covering PEP 768 remote debugging with sys.remote_exec()
- [sys.remote_exec](https://docs.python.org/3.14/library/sys.html#sys.remote_exec) — Official documentation for sys.remote_exec() introduced in Python 3.14

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*