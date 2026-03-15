---
source_course: "paperclip-setup"
source_lesson: "paperclip-setup-first-company"
---

# First Company Setup

## Introduction

Once Paperclip is running, the next step is creating your first company. A company in Paperclip is the organizational container that holds agents, issues, secrets, and configuration. This lesson walks through the web UI workflow for setting up a company from scratch.

## Key Concepts

- **Company**: The top-level organizational unit in Paperclip that contains all agents, issues, and configuration.
- **CEO Agent**: The root agent in the organizational hierarchy that delegates work to other agents.
- **Adapter**: The AI model backend that powers an agent (e.g., `claude_local`, `codex_local`, `openai`).
- **Org Chart**: The hierarchical structure of agents within a company, defining reporting lines and delegation.

## Real World Context

Think of a Paperclip company as a virtual software team. Just as a real company has a CEO who sets direction, managers who break down work, and engineers who write code, a Paperclip company mirrors this structure with AI agents. The CEO agent receives high-level objectives and delegates tasks through the org chart to specialized agents, each configured with the right model adapter and budget for their role.

## Deep Dive

Open the Paperclip UI at `http://localhost:3100` in your browser. The first screen prompts you to create a company.

Start by defining the company identity. Enter a name, a mission statement that describes what the company works on, and goals that specify current priorities. These are not just labels — agents use the mission and goals to make autonomous decisions about task prioritization.

```bash
# You can also create a company via the CLI
pnpm paperclipai company create --name "My Project" \
  --mission "Build and maintain the analytics dashboard" \
  --goals "Ship v2.0 by end of quarter"
```

Next, create the CEO agent. Choose an adapter that determines which AI model the agent uses. Common adapters include `claude_local` for Anthropic's Claude, `codex_local` for OpenAI Codex, and others depending on your available API keys.

Set a budget for the CEO agent. The budget controls how many tokens or API calls the agent can consume per task or per day. Start conservative and increase as you build confidence in the agent's behavior.

```bash
# Create the CEO agent via CLI
pnpm paperclipai agent create --company-id <id> \
  --name "CEO" \
  --adapter claude_local \
  --budget-daily 100000
```

After the CEO is created, add additional agents for specialized roles. Each agent gets its own adapter, budget, and position in the org chart. The org chart determines which agents can delegate to which — a manager agent can assign tasks to its reports but not to agents outside its subtree.

Finally, review the org chart in the web UI to verify the hierarchy makes sense. Agents without a manager report directly to the CEO.

## Common Pitfalls

1. **Setting budgets too high initially** — Agents with unlimited budgets can burn through API credits quickly during testing. Start with conservative daily limits and scale up after observing behavior.
2. **Choosing the wrong adapter** — Each adapter has different capabilities and costs. Using a powerful model like Claude for simple formatting tasks wastes budget. Match the adapter to the agent's role complexity.

## Best Practices

1. **Write specific mission and goals** — Vague missions like "do stuff" produce poor agent decisions. Be concrete: "Maintain the React frontend for the billing dashboard" gives agents the context they need.
2. **Start with a flat org chart** — Begin with a CEO and 2-3 direct report agents. Add hierarchy only when you observe the CEO struggling to manage too many direct tasks.

## Summary

- A company is Paperclip's organizational container for agents and work.
- Set a clear mission and goals — agents use these for decision-making.
- The CEO agent sits at the top of the org chart and delegates to other agents.
- Each agent has an adapter (AI model) and budget.
- Start with conservative budgets and a flat hierarchy, then expand.

## Code Examples

**Creating a company and CEO agent using the Paperclip CLI**

```bash
# Create company via CLI
pnpm paperclipai company create --name "My Project" \
  --mission "Build and maintain the analytics dashboard"

# Create CEO agent
pnpm paperclipai agent create --company-id <id> \
  --name "CEO" --adapter claude_local
```


## Resources

- [Paperclip Company Setup](https://paperclip.ing/docs) — Guide to creating and configuring companies in Paperclip

---

> 📘 *This lesson is part of the [Paperclip Setup & Deployment](https://stanza.dev/courses/paperclip-setup) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*