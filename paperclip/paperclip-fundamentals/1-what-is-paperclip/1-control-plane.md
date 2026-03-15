---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-control-plane"
---

# The Control Plane for AI Companies

## Introduction

Managing multiple AI agents is chaos. You end up with tab sprawl across dozens of terminals, no shared context between agents, runaway costs nobody tracks, and zero accountability for what each agent actually did. Paperclip solves this by acting as the company OS for autonomous AI organizations.

## Key Concepts

- **Control Plane**: The orchestration layer that manages agents, assigns work, tracks costs, and enforces governance — without executing the work itself
- **Execution Plane**: The layer where agents actually run code, call APIs, and produce output — handled by adapters, not Paperclip
- **Company OS**: Paperclip treats your fleet of AI agents like a real company with employees, managers, budgets, and org charts
- **Self-Hosted**: Paperclip runs on your infrastructure, keeping all data under your control

## Real World Context

Imagine you have five AI coding agents working on a project. One is writing backend code, another is handling frontend, a third is writing tests, and two more are doing code review. Without a control plane, these agents step on each other's work, duplicate effort, and burn through your API budget with no oversight. Paperclip gives you the structure to coordinate them like a real engineering team.

## Deep Dive

The core insight behind Paperclip is a simple analogy: if an AI coding agent like OpenClaw is an employee, then Paperclip is the company. It provides the organizational structure, policies, and management layer that make individual agents productive as a team.

Paperclip is open-source under the MIT license, self-hosted, and built on Node.js 20+.

Once running, Paperclip serves a web UI and REST API on port 3100. The UI gives you a dashboard to manage your AI company, while the API lets agents and scripts interact programmatically.

The distinction between control plane and execution plane is fundamental. Paperclip never runs your agents' code directly. Instead, it orchestrates through adapters — bridges to agent runtimes like Claude, Codex, or Gemini. When an agent needs to do work, Paperclip tells the adapter to spawn a process, passes context, and collects results.

```bash
# Paperclip's default URL
# UI:  http://localhost:3100
# API: http://localhost:3100/api
```

This separation means Paperclip stays lightweight and runtime-agnostic. You can swap agent runtimes without changing your organizational structure, budgets, or governance rules.

Paperclip uses PGlite as an embedded database for local development, so there is no need to set up PostgreSQL just to get started. Everything runs in a single process on your machine.

## Common Pitfalls

1. **Thinking Paperclip runs your agents** — Paperclip orchestrates agents through adapters. The actual code execution happens in the adapter's runtime, not inside Paperclip.
2. **Using Paperclip for a single agent** — Paperclip is designed for multi-agent coordination. If you only have one agent, the overhead of a control plane is not justified.

## Best Practices

1. **Start with the default setup** — Use PGlite and the built-in adapters to get familiar before customizing. The defaults are production-ready for small teams.
2. **Treat the control plane as the source of truth** — All work assignments, status updates, and cost tracking should flow through Paperclip, not through side channels.

## Summary

- Paperclip is the control plane for autonomous AI companies, orchestrating agents without executing their work
- It is open-source (MIT), self-hosted, and runs on Node.js 20+ with PGlite for zero-config local development
- The server runs on port 3100, providing both a web UI and REST API
- Control plane vs execution plane: Paperclip manages, adapters execute
- Designed for multi-agent coordination, not single-agent use cases

## Resources

- [Paperclip Official Documentation](https://paperclip.ing/docs) — Official documentation for the Paperclip control plane
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Source code and README for Paperclip

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*