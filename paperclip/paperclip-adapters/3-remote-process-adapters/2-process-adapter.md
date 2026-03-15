---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-process-adapter"
---

# Process Adapter

## Introduction

The process adapter (`process`) is Paperclip's most flexible adapter. It executes arbitrary shell commands, making it suitable for script-based automation, batch processing, and any task that can be expressed as a command-line invocation. If you can run it in a terminal, the process adapter can run it as a Paperclip agent.

## Key Concepts

- **Adapter Type**: `process` — executes arbitrary shell commands as agent tasks
- **Command and Args**: The shell command and its arguments, configured as separate fields for clarity
- **Environment Injection**: Paperclip environment variables are merged with custom variables and passed to the process
- **stdout Capture**: The adapter captures the command's stdout and parses it for results
- **Security Controls**: Sandboxing and access controls limit what the process can do

## Real World Context

A data engineering team uses Paperclip to orchestrate nightly ETL jobs. Each job is a Python script that extracts data from an API, transforms it, and loads it into a database. Instead of building a custom adapter for Python, the team uses the process adapter to run their scripts directly. Paperclip handles scheduling via heartbeats, and the process adapter handles execution.

## Deep Dive

The process adapter configuration specifies the command, arguments, working directory, environment, and timeouts:

```json
{
  "adapterType": "process",
  "adapterConfig": {
    "command": "python3",
    "args": ["scripts/analyze.py", "--output", "json"],
    "cwd": "/home/dev/data-pipeline",
    "env": {
      "DATABASE_URL": "postgresql://localhost:5432/analytics",
      "API_TOKEN": {
        "type": "secret_ref",
        "secretId": "data-api-token",
        "version": "latest"
      }
    },
    "timeoutSec": 900,
    "graceSec": 15
  }
}
```

The `command` field is the executable to run. The `args` array contains the arguments passed to it. Together, this is equivalent to running `python3 scripts/analyze.py --output json` in the terminal.

The `env` block merges with the Paperclip environment variables. Your process receives both the custom variables you define and the standard Paperclip context variables (run ID, task ID, etc.):

```typescript
// Inside the process adapter's execute function
const paperclipEnv = buildPaperclipEnv(ctx.agent);
const processEnv = {
  ...process.env,
  ...paperclipEnv,
  ...resolvedCustomEnv
};

const result = await runChildProcess({
  command: config.command,
  args: config.args,
  cwd: config.cwd,
  env: processEnv,
  timeoutMs: config.timeoutSec * 1000,
  graceMs: config.graceSec * 1000
});
```

The adapter captures stdout from the process and attempts to parse it as JSON. If the output contains a valid `AdapterExecutionResult` structure, Paperclip uses it directly. Otherwise, the raw stdout becomes the run summary.

Timeout handling works in two phases. After `timeoutSec`, Paperclip sends SIGTERM to give the process a chance to clean up. After an additional `graceSec`, it sends SIGKILL to force termination.

## Common Pitfalls

- **Forgetting to set the correct cwd**: If your script uses relative file paths, the working directory must be set correctly or the script will fail to find its dependencies.
- **Not making the command executable**: On Unix systems, scripts need execute permissions (`chmod +x`). Without them, the process adapter gets a "permission denied" error.
- **Ignoring exit codes**: A process that exits with code 0 is treated as successful. Non-zero exit codes signal failure. Ensure your scripts use appropriate exit codes.

## Best Practices

- **Output structured JSON from your scripts**: If your script outputs a JSON object matching AdapterExecutionResult, Paperclip can track usage and costs automatically.
- **Use graceSec for cleanup**: Set `graceSec` to give your process enough time to close database connections, flush buffers, and clean up temporary files.
- **Limit process capabilities**: Run processes under restricted user accounts and avoid giving them root access. Use environment variables for secrets instead of command-line arguments.

## Summary

- The `process` adapter executes arbitrary shell commands as Paperclip agent tasks
- Configuration includes command, args, cwd, environment variables, and timeout settings
- Paperclip environment variables are merged with custom variables and passed to the process
- stdout is captured and optionally parsed as JSON for structured result reporting
- Two-phase timeout (SIGTERM then SIGKILL) allows graceful shutdown

## Code Examples

**Process adapter configuration for running a Python script with custom environment, working directory, and two-phase timeout**

```json
{
  "command": "python3",
  "args": ["scripts/analyze.py", "--output", "json"],
  "cwd": "/home/dev/data-pipeline",
  "env": {"DATABASE_URL": "postgresql://localhost:5432/analytics"},
  "timeoutSec": 900,
  "graceSec": 15
}
```


## Resources

- [Paperclip Process Adapter Documentation](https://paperclip.ing/docs/adapters/process) — Official documentation for the process adapter configuration and usage
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Source code for the process adapter implementation

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*