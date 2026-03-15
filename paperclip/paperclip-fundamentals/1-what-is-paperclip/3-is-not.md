---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-what-paperclip-is-not"
---

# What Paperclip Is Not

## Introduction

Paperclip occupies a specific niche in the AI tooling ecosystem. To use it effectively, you need to understand not just what it does, but what it deliberately does not do. This lesson draws clear boundaries around Paperclip's scope and helps you decide when it is the right tool.

## Key Concepts

- **Not a Chatbot**: Paperclip does not provide a conversational interface for end users
- **Not an Agent Framework**: Paperclip does not help you build agents — it manages agents you have already built or configured
- **Not a Workflow Builder**: There are no drag-and-drop pipelines or visual DAGs
- **Not for Single-Agent Use**: The overhead of a control plane is not justified for one agent

## Real World Context

A developer discovers Paperclip and thinks it is like LangChain or CrewAI — a framework for building agents. They try to define agent logic inside Paperclip and get frustrated when they cannot find prompt engineering or tool definition features. The confusion stems from a category error. Paperclip is to agents what a company's HR and operations department is to employees: it manages them, it does not create them.

## Deep Dive

Let us walk through each misconception in detail.

**Paperclip is not a chatbot.** It has no end-user chat interface. The UI is an admin dashboard for managing companies, agents, and work. Your users never interact with Paperclip directly — they interact with your agents through whatever interfaces your agents expose.

**Paperclip is not an agent framework.** Frameworks like LangChain, CrewAI, or AutoGen help you build agents by providing abstractions for prompts, tools, memory, and chains. Paperclip does none of this. Instead, it expects agents to already exist as runtimes (like Claude, Codex, or Gemini) and manages them through adapters.

```bash
# Paperclip's 7 built-in adapters connect to existing agent runtimes
# claude_local  — Anthropic Claude (local CLI)
# codex_local   — OpenAI Codex (local CLI)
# gemini_local  — Google Gemini (local CLI)
# opencode_local — OpenCode (local CLI)
# openclaw      — OpenClaw agent runtime
# process       — Generic process spawner
# http          — HTTP-based agent endpoint
```

Each adapter bridges Paperclip to an existing runtime. Paperclip tells the adapter what work to do, the adapter spawns the agent process, and the agent does the actual execution. Paperclip never touches the agent's internals.

**Paperclip is not a workflow builder.** Tools like n8n, Temporal, or Apache Airflow let you define directed acyclic graphs (DAGs) of tasks with conditional branching. Paperclip has no visual workflow editor and no DAG execution engine. Work is organized through issues in a backlog, and agents pick up issues through heartbeats — more like a Kanban board than a pipeline.

**Paperclip is not for single-agent use.** If you have one AI agent doing one thing, you do not need a control plane. Paperclip's value comes from coordinating multiple agents: preventing duplicate work, managing budgets across agents, enforcing an org chart, and providing governance. A single agent can just run directly.

So when should you use Paperclip? Use it when you have:

```bash
# Good fit for Paperclip:
# ✓ Multiple AI agents that need coordination
# ✓ Work that needs to be tracked and assigned
# ✓ Budgets that need enforcement across agents
# ✓ Humans who need oversight over agent decisions
# ✓ A need for audit trails of agent activity

# Not a good fit:
# ✗ Single chatbot application
# ✗ Building an agent from scratch
# ✗ Visual workflow automation
# ✗ One-off scripting tasks
```

Paperclip shines when the problem is organizational, not technical. If your challenge is "how do I coordinate five agents working on the same codebase without them stepping on each other," Paperclip is exactly the right tool.

## Common Pitfalls

1. **Trying to define agent prompts in Paperclip** — Agent behavior is configured in the adapter or runtime, not in Paperclip. Paperclip manages work assignments and organizational structure.
2. **Comparing Paperclip to CrewAI or LangGraph** — These are agent frameworks. Paperclip is a management layer. They solve different problems and can be used together.

## Best Practices

1. **Use Paperclip alongside your agent framework** — Build your agents with whatever framework you prefer, then use Paperclip to coordinate them at the organizational level.
2. **Evaluate whether you need multi-agent coordination** — If you are running a single agent, skip Paperclip and run the agent directly. Add Paperclip when coordination becomes the bottleneck.

## Summary

- Paperclip is not a chatbot, agent framework, workflow builder, or single-agent tool
- It manages existing agents through adapters — 7 built-in adapters connect to runtimes like Claude, Codex, Gemini, and OpenClaw
- Use Paperclip when you need multi-agent coordination, budget enforcement, org charts, and governance
- Paperclip complements agent frameworks rather than replacing them
- The value is organizational: preventing duplicate work, enforcing structure, and providing oversight

## Code Examples

**The 7 built-in adapters that connect Paperclip to agent runtimes**

```bash
# Paperclip adapters:
# claude_local, codex_local, gemini_local,
# opencode_local, openclaw, process, http
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Source code, README, and adapter documentation

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*