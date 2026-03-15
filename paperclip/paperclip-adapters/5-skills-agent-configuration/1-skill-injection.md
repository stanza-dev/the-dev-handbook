---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-skill-injection"
---

# Runtime Skill Injection

## Introduction

Skills in Paperclip teach agents how to perform specific workflows without retraining the underlying model. Instead of fine-tuning, skills inject structured instructions that guide the agent's behavior at runtime. There are four approaches to skill injection, each suited to different runtimes and deployment models.

## Key Concepts

- **Skills**: Structured instructions that teach agents workflows without retraining — think of them as runtime plugins for agent behavior
- **Temp Directory + Symlink**: The preferred approach — create a temporary directory, symlink skill files into it, pass the path to the runtime via a CLI flag, and clean up after execution
- **Global Plugins**: Install skills as global plugins that the runtime discovers automatically
- **Environment Variable**: Pass skill paths or content via environment variables
- **Embedded in Prompts**: Inline skill instructions directly in the prompt template

## Real World Context

A team has built a Paperclip skill that teaches agents how to follow their company's Git workflow: create a feature branch, commit with conventional commit messages, and open a pull request with the correct template. Instead of explaining this in every prompt, they inject the skill at runtime. The agent discovers the skill and follows the workflow automatically, regardless of which adapter or model is being used.

## Deep Dive

Skill injection is the mechanism by which Paperclip makes skills discoverable to agent runtimes. Different runtimes have different plugin systems, so Paperclip supports four approaches.

### Approach 1: Temp Directory + Symlink (Preferred)

This is Paperclip's preferred approach because it is clean, isolated, and works with any runtime that accepts a skills directory path:

```typescript
import { mkdtemp, symlink, rm } from "fs/promises";
import { join } from "path";
import { tmpdir } from "os";

async function injectSkills(skillPaths: string[], runtimeArgs: string[]): Promise<() => Promise<void>> {
  // Step 1: Create temp directory
  const tempDir = await mkdtemp(join(tmpdir(), "paperclip-skills-"));

  // Step 2: Symlink each skill into the temp dir
  for (const skillPath of skillPaths) {
    const skillName = basename(skillPath);
    await symlink(skillPath, join(tempDir, skillName));
  }

  // Step 3: Pass temp dir via CLI flag
  runtimeArgs.push("--skills-dir", tempDir);

  // Step 4: Return cleanup function
  return async () => {
    await rm(tempDir, { recursive: true, force: true });
  };
}
```

The flow is: create a temporary directory, symlink the skill files into it, pass the directory path to the runtime via a CLI flag (like `--skills-dir`), let the agent run, then clean up the temp directory. Symlinks are used instead of copies so that skill updates take effect immediately.

Here is how the preferred approach integrates into the execute function:

```typescript
export async function execute(ctx: AdapterExecutionContext): Promise<AdapterExecutionResult> {
  const cleanup = await injectSkills(ctx.skills, runtimeArgs);
  try {
    const result = await runChildProcess({ command, args: runtimeArgs, env, cwd });
    return parseResult(result);
  } finally {
    await cleanup();
  }
}
```

The `try/finally` pattern ensures cleanup runs even if the execution fails.

### Approach 2: Global Plugins

Some runtimes support global plugin directories. Skills can be installed there so the runtime discovers them automatically:

```typescript
// Install skill as global plugin
await copySkillToGlobalPluginDir(skill, "~/.runtime/plugins/");
```

This approach is simpler but has drawbacks: global plugins affect all runs, and cleanup is harder to manage.

### Approach 3: Environment Variable

Pass skill information through environment variables:

```typescript
const env = {
  ...paperclipEnv,
  SKILLS_PATH: skillPaths.join(":"),
  SKILL_COUNT: String(skillPaths.length)
};
```

This works when the runtime reads skill configuration from environment variables. It is less common but useful for runtimes with limited CLI options.

### Approach 4: Embedded in Prompts

Inline skill instructions directly in the prompt template:

```typescript
const prompt = `${basePrompt}\n\n## Skills\n${skillContent}`;
```

This is the simplest approach but scales poorly — long skill documents consume token budget and increase costs.

## Common Pitfalls

- **Forgetting cleanup with the temp directory approach**: If the cleanup function is not called (e.g., due to an unhandled exception), temp directories accumulate and consume disk space.
- **Using global plugins in multi-agent setups**: Global plugins affect all agents using the same runtime. This can cause skill conflicts when different agents need different skill sets.
- **Embedding large skills in prompts**: Inlining skill content consumes tokens on every run. A 5,000-token skill adds significant cost over hundreds of runs.

## Best Practices

- **Use the temp directory + symlink approach by default**: It provides clean isolation, automatic cleanup, and works with any runtime that accepts a directory path.
- **Use try/finally for cleanup**: Always wrap execution in try/finally to ensure cleanup runs regardless of success or failure.
- **Version your skills**: Track skill files in version control so you can roll back to previous versions if a skill change causes agent regressions.

## Summary

- Skills teach agents workflows without retraining — they are runtime plugins for behavior
- Four injection approaches: temp dir + symlink (preferred), global plugins, env var, embedded in prompts
- The preferred approach creates a temp dir, symlinks skills, passes the path via CLI flag, and cleans up
- Always use try/finally to ensure cleanup runs even when execution fails
- Global plugins and embedded prompts have scaling drawbacks that make the temp dir approach preferable

## Code Examples

**The preferred skill injection approach: create temp dir, symlink skills, pass via CLI flag, return cleanup function**

```typescript
async function injectSkills(skillPaths: string[], runtimeArgs: string[]): Promise<() => Promise<void>> {
  const tempDir = await mkdtemp(join(tmpdir(), "paperclip-skills-"));
  for (const skillPath of skillPaths) {
    await symlink(skillPath, join(tempDir, basename(skillPath)));
  }
  runtimeArgs.push("--skills-dir", tempDir);
  return async () => { await rm(tempDir, { recursive: true, force: true }); };
}
```


## Resources

- [Paperclip Skills Documentation](https://paperclip.ing/docs/skills) — Official documentation for creating and injecting skills into Paperclip agents
- [Paperclip GitHub — Skill Injection](https://github.com/paperclipai/paperclip) — Source code for skill injection utilities used by built-in adapters

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*