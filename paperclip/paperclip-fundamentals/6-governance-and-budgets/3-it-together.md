---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-putting-it-together"
---

# Putting It All Together

## Introduction

You have now learned each of Paperclip's five pillars individually. This lesson connects them into a complete end-to-end workflow — from creating a company to governing agent decisions. Understanding how the pillars interact is what turns knowledge of individual features into the ability to run an AI company.

## Key Concepts

- **End-to-End Flow**: Create company → set goals → hire agents → assign work → monitor costs → govern
- **Pillar Interactions**: Each pillar depends on and enhances the others
- **Scaling Patterns**: Common approaches for growing an AI company from a few agents to many

## Real World Context

Think of launching a startup. Day one: you incorporate the company (create company) and define your mission (set goals). Week one: you hire your first employees (hire agents) and give them projects (create projects and issues). Ongoing: you track expenses (monitor costs) and the board reviews major decisions (governance). Paperclip automates this entire lifecycle for AI agent organizations.

## Deep Dive

Here is the complete end-to-end workflow for setting up and running an AI company with Paperclip:

**Step 1: Create the Company**

```bash
# Create the company with a monthly budget
curl -X POST http://localhost:3100/api/companies \
  -H "Content-Type: application/json" \
  -d '{
    "name": "DevBot Corp",
    "description": "AI-powered development team for SaaS applications",
    "monthlyBudget": 100000
  }'
```

This establishes the organizational container. The $1,000 monthly budget sets the aggregate spending limit for all agents.

**Step 2: Define Goals**

```bash
# Set a company-level goal
curl -X POST http://localhost:3100/api/companies/{companyId}/goals \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Launch SaaS MVP by April",
    "description": "Build and deploy a functional MVP with auth, dashboard, and billing",
    "level": "company"
  }'
```

Goals give agents strategic context. Every issue they work on traces back to this goal.

**Step 3: Hire Agents (via governance)**

Agent hiring goes through the board approval process. You propose hires, board members review, and approved agents are created in the org chart.

**Step 4: Create Projects and Issues**

Projects represent deliverables. Issues are the atomic tasks. The hierarchy is: company → goals → projects → issues → agents.

**Step 5: Agents Execute via Heartbeats**

Agents wake on schedule, review their assignments, checkout issues, execute work via their adapters, and report results. The cycle repeats.

**Step 6: Monitor Costs**

Cost reports at company, agent, and project levels show where money is going. The 80% soft alert and 100% hard stop prevent overspending.

**Step 7: Govern Critical Decisions**

Board members review and approve hiring proposals, strategy changes, and other critical actions.

The pillars interact in important ways:

```bash
# How pillars interact:
# Companies → contain Goals, Agents, Projects
# Goals     → guide Projects and Issues
# Agents    → organized in org chart, consume budgets
# Issues    → assigned to Agents, tracked through lifecycle
# Heartbeats → execute Issues via Adapters
# Governance → approves Hiring, controls Strategy
# Budgets   → limit Agent spending, reset monthly
```

Common scaling patterns include starting with a small team (CEO + 2-3 ICs), adding managers when you hit 5+ ICs, using multiple companies for separate projects or clients, and gradually increasing budgets based on observed cost patterns.

The key insight is that Paperclip is not just a collection of features — it is a system where each part reinforces the others. Governance ensures agents are hired thoughtfully. Budgets ensure agents do not overspend. The org chart ensures clear accountability. Issues ensure work is tracked. Heartbeats ensure work gets done.

## Common Pitfalls

1. **Setting up all pillars at once** — Start simple: one company, one goal, a few agents. Add complexity as you learn the system.
2. **Ignoring governance early on** — Even with just two agents, use governance for hiring. It establishes the habit and provides an audit trail from day one.

## Best Practices

1. **Follow the workflow order** — Company first, then goals, then agents, then projects and issues. Each step depends on the previous ones.
2. **Review cost reports weekly** — Do not wait for the 80% alert. Regular cost reviews help you optimize agent efficiency proactively.

## Summary

- The end-to-end flow: create company → set goals → hire agents → assign work → monitor costs → govern
- All five pillars interact and reinforce each other
- Start small and scale: CEO + 2-3 ICs, add managers at 5+ ICs
- Use governance from day one for accountability and audit trails
- Regular cost monitoring prevents surprises and optimizes efficiency

## Code Examples

**The end-to-end API call sequence for setting up an AI company**

```bash
# Complete setup flow:
# 1. POST /api/companies
# 2. POST /api/companies/{id}/goals
# 3. POST /api/companies/{id}/agent-hires
# 4. POST /api/companies/{id}/projects
# 5. POST /api/issues
# 6. Monitor: GET /api/companies/{id}/costs
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation — the complete reference
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Source code, examples, and community resources

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*