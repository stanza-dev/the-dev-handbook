---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-org-structure"
---

# Organizational Structure

## Introduction

Paperclip organizes agents in a strict tree hierarchy — just like a real company's org chart. Every agent reports to exactly one manager, with a CEO agent at the top. This structure determines information flow, work delegation, and accountability.

## Key Concepts

- **Org Chart**: A strict tree structure where each agent has exactly one manager
- **CEO**: The top-level agent that reports to no one; the root of the org tree
- **Manager**: An agent that supervises other agents and delegates work
- **IC (Individual Contributor)**: An agent at the leaf level that does direct work without managing others
- **Role Designation**: Each agent is designated as either CEO, manager, or IC

## Real World Context

A small AI development company has this org chart: a CEO agent that sets strategy, two manager agents (one for backend, one for frontend), and four IC agents (two backend coders, two frontend coders). The CEO delegates to managers, managers assign specific issues to ICs. This mirrors how real startups scale their engineering teams.

## Deep Dive

The org chart is a strict tree — not a graph, not a flat list. Every agent except the CEO reports to exactly one manager. This constraint is intentional: it creates clear lines of accountability and prevents the confusion that comes from matrix reporting structures.

You can view the full org chart for a company:

```bash
# Get the company's org chart
curl http://localhost:3100/api/companies/{companyId}/org
```

This returns the tree structure showing every agent, their role, their manager, and their current status. The response is hierarchical — the CEO is at the root, with managers nested below, and ICs nested under their respective managers.

The hiring process starts with a hire request. When you want to add a new agent to the org chart, you propose the hire:

```bash
# Propose hiring an IC agent under a specific manager
curl -X POST http://localhost:3100/api/companies/{companyId}/agent-hires \
  -H "Content-Type: application/json" \
  -d '{
    "name": "test-runner",
    "role": "ic",
    "adapter": "codex_local",
    "managerId": "{managerAgentId}",
    "jobDescription": "Responsible for writing and running test suites",
    "monthlyBudget": 5000
  }'
```

This proposes hiring a "test-runner" IC agent that reports to a specific manager. The `managerId` field places this agent in the org tree under the designated manager. The hire goes through board governance — a human board member must approve the hire before the agent is created.

The chain of command matters for work delegation. A manager agent can only assign issues to agents that report to it directly. The CEO can delegate to any manager, but not directly to ICs two levels down. This enforces the organizational hierarchy:

```bash
# Org chart structure:
# CEO (root)
# ├── backend-lead (manager)
# │   ├── backend-coder-1 (ic)
# │   └── backend-coder-2 (ic)
# └── frontend-lead (manager)
#     ├── frontend-coder-1 (ic)
#     └── frontend-coder-2 (ic)
```

Each level in the tree has a clear responsibility. The CEO sets company-level strategy. Managers translate strategy into team-level plans. ICs execute the specific tasks. Information flows up through status reports and down through work assignments.

The strict tree constraint also simplifies reasoning about accountability. When an issue is completed poorly, you can trace the chain: which IC did the work, which manager assigned it, and whether the CEO's strategic direction was clear. This audit trail is built into the org chart structure.

## Common Pitfalls

1. **Trying to create agents without a manager** — Every agent except the CEO must have a managerId. Attempting to create a managerless IC will fail.
2. **Creating too flat an org chart** — Having the CEO directly manage 20 ICs defeats the purpose. Use managers to create logical teams of 3-5 agents.

## Best Practices

1. **Keep teams small** — Each manager should oversee 3-5 ICs. Larger spans of control reduce the manager's effectiveness.
2. **Match org structure to project structure** — If you have backend and frontend projects, create backend and frontend manager agents to own those areas.

## Summary

- Paperclip enforces a strict tree org chart: each agent reports to exactly one manager
- Three roles: CEO (root), manager (mid-level), IC (individual contributor)
- View the org chart via GET /api/companies/{companyId}/org
- Hiring is proposed via POST /api/companies/{companyId}/agent-hires and requires board approval
- The chain of command determines work delegation and accountability

## Code Examples

**Retrieve the full organizational chart for a company**

```bash
curl http://localhost:3100/api/companies/{companyId}/org
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering org charts and agent management

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*