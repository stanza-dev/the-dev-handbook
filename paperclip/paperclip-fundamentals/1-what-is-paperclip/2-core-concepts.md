---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-core-concepts"
---

# Core Concepts Overview

## Introduction

Paperclip is built on five interconnected pillars that mirror the structure of a real company. Understanding how these pillars fit together is essential before diving into any individual feature. This lesson maps out the full context chain from company to agent execution.

## Key Concepts

- **Company**: The top-level organizational unit with a name, description, status, and monthly budget
- **Agent**: An AI employee with a role, reporting line, budget, and adapter configuration
- **Issue**: An atomic unit of work that agents check out, execute, and complete
- **Heartbeat**: A scheduled execution cycle where an agent wakes up, reviews work, and acts
- **Governance**: A board-of-directors model where humans approve critical decisions like hiring agents or setting strategy

## Real World Context

Think of Paperclip like setting up a small startup. You create the company (Company), define what you want to achieve (Goals), break that into projects (Projects), create tasks (Issues), hire people (Agents), have them show up for work regularly (Heartbeats), and make sure a board approves big decisions (Governance). Paperclip digitizes this entire flow for AI agents.

## Deep Dive

The five pillars connect through a clear hierarchy. At the top sits the Company. A company defines Goals, which represent strategic objectives at three levels: company-wide, team-level, and agent-level. Goals answer the question "why are we doing this?"

Goals feed into Projects, which answer "what are we building?" Each project has a workspace that connects it to a code repository or development environment. Projects contain Issues — the atomic tasks that agents actually work on.

Here is how the context chain flows:

```bash
# The Paperclip context chain
# Company → Goals → Projects → Issues → Agents
#
# Example:
# Company: "Acme AI Dev Shop"
#   └── Goal: "Ship MVP by Q2"
#       └── Project: "Backend API"
#           └── Issue: "Implement user authentication"
#               └── Agent: "backend-coder" (checks out and executes)
```

This hierarchy ensures every piece of work traces back to a strategic objective. When an agent checks out an issue, it receives the full context chain — the issue description, the project it belongs to, the goal it serves, and the company it is part of.

Agents are organized in an org chart with a strict tree structure. Each agent reports to exactly one manager. The CEO agent sits at the top. Below the CEO are manager agents who coordinate teams of individual contributor (IC) agents.

Heartbeats are the execution mechanism. An agent does not run continuously — it wakes up on a schedule, reviews its assigned issues, checks out work, executes through its adapter, and reports results. Between heartbeats, the agent's state is persisted so context carries over.

Governance provides human oversight. A board of directors (humans) reviews and approves critical actions like hiring new agents or changing the CEO's strategy. This prevents autonomous runaway.

Paperclip supports multiple companies within a single instance. You can run several AI companies side by side, each with its own org chart, goals, projects, and budgets.

```bash
# List all companies via the API
curl http://localhost:3100/api/companies

# Get a specific company's org chart
curl http://localhost:3100/api/companies/{companyId}/org
```

These API calls demonstrate how Paperclip exposes the organizational structure programmatically. Every concept in the hierarchy is accessible through the REST API.

## Common Pitfalls

1. **Skipping goals and jumping to issues** — Without goals, your agents have no strategic context. Always define at least a company-level goal before creating issues.
2. **Ignoring the org chart** — The reporting structure is not decorative. It determines who can assign work to whom and how information flows.

## Best Practices

1. **Define the context chain top-down** — Start with the company, then goals, then projects, then issues. This ensures every task has strategic alignment.
2. **Use governance for irreversible actions** — Agent hiring and CEO strategy changes should always go through board approval.

## Summary

- Paperclip has five pillars: Companies, Agents, Issues, Heartbeats, and Governance
- The context chain flows: Company → Goals → Projects → Issues → Agents
- Each agent reports to exactly one manager in a strict tree org chart
- Heartbeats are scheduled execution cycles, not continuous running
- Governance provides human oversight through a board-of-directors model
- Multiple companies can run in a single Paperclip instance

## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering all core concepts

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*