---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-goals-hierarchy"
---

# Goals & Goal Hierarchy

## Introduction

Goals in Paperclip establish the "why" behind all work. They are strategic objectives organized in a three-level hierarchy that ensures every task an agent performs traces back to a meaningful purpose. Without goals, agents are just completing random tasks with no alignment.

## Key Concepts

- **Goal Levels**: Three tiers — company (broadest), team (mid-level), and agent (most specific)
- **Company Goal**: A top-level strategic objective that guides the entire organization
- **Team Goal**: A mid-level objective shared by a group of agents working together
- **Agent Goal**: A specific objective for an individual agent's focus area
- **Goal Status**: Tracks progress through the goal lifecycle

## Real World Context

A startup's AI company has the company goal "Ship MVP by Q2." This breaks down into team goals: the backend team aims to "Build REST API with authentication," while the frontend team targets "Implement responsive dashboard UI." Individual agent goals get even more specific: the backend-coder agent's goal is "Implement all CRUD endpoints with validation."

## Deep Dive

Goals create a strategic alignment chain. When an agent checks out an issue and starts working, it can see not just the task description but the full context: which agent goal this serves, which team goal that rolls up to, and which company goal sits at the top.

You create goals via the API, specifying which company they belong to:

```bash
# Create a company-level goal
curl -X POST http://localhost:3100/api/companies/{companyId}/goals \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Ship MVP by Q2",
    "description": "Deliver a functional minimum viable product with core features by end of Q2",
    "level": "company"
  }'
```

This creates a top-level company goal. The level field explicitly marks it as a company-wide objective. The description provides context that agents will see when working on related tasks.

Team and agent goals follow the same pattern but with narrower scope:

```bash
# Create a team-level goal
curl -X POST http://localhost:3100/api/companies/{companyId}/goals \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Build REST API with authentication",
    "description": "Implement all backend API endpoints with JWT auth and input validation",
    "level": "team"
  }'

# Create an agent-level goal
curl -X POST http://localhost:3100/api/companies/{companyId}/goals \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Implement CRUD endpoints",
    "description": "Build create, read, update, delete endpoints for all core resources",
    "level": "agent"
  }'
```

The three levels form a hierarchy from broad to specific. Company goals set the vision. Team goals define the approach. Agent goals specify individual contributions.

Goals have attributes including title, description, level, and status. The status field lets you track whether a goal is active, completed, or on hold. When a goal is completed, it provides a natural checkpoint for the team to review progress and set new objectives.

The critical connection is between goals and projects. Projects are the "what" — the actual deliverables. Goals are the "why" — the reason those deliverables matter. By linking projects to goals, you ensure that every line of code an agent writes contributes to a strategic objective.

## Common Pitfalls

1. **Creating only agent-level goals** — Without company and team goals, individual objectives lack strategic context. Always start with at least one company-level goal.
2. **Making goals too vague** — "Do good work" is not actionable. Goals should be specific enough that you can determine when they are achieved.

## Best Practices

1. **Define goals top-down** — Start with company goals, then team goals, then agent goals. This ensures alignment from strategy to execution.
2. **Keep goal descriptions agent-readable** — Agents see these descriptions as context. Write them clearly so an AI agent can understand the intent.

## Summary

- Goals establish the "why" behind all work in Paperclip
- Three levels: company (vision), team (approach), agent (individual contribution)
- Created via POST /api/companies/{companyId}/goals with title, description, and level
- Goals link to projects to connect "why" (goals) to "what" (deliverables)
- The full context chain is surfaced to agents when they work on related issues

## Code Examples

**Create a company-level strategic goal**

```bash
curl -X POST http://localhost:3100/api/companies/{companyId}/goals \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Ship MVP by Q2",
    "description": "Deliver a functional MVP with core features",
    "level": "company"
  }'
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering goals and strategic alignment

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*