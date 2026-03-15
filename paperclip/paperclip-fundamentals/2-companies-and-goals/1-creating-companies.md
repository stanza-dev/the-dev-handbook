---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-creating-companies"
---

# Creating and Managing Companies

## Introduction

The Company is the top-level organizational unit in Paperclip. Everything else — goals, projects, agents, issues — lives inside a company. Before you can hire AI agents or assign work, you need to create and configure at least one company.

## Key Concepts

- **Company**: A named organizational unit with a description, status, and monthly budget
- **Status Lifecycle**: Companies move through three statuses — active, paused, and archived
- **Monthly Budget**: Set in cents (not dollars) to avoid floating-point issues; controls aggregate spend across all agents
- **Multi-Company**: A single Paperclip instance can manage multiple independent companies

## Real World Context

A software consultancy uses Paperclip to manage AI agent teams for three different clients. Each client gets their own company in Paperclip with separate budgets, goals, and agent teams. When a client project wraps up, the company is archived — preserving the history without cluttering the active dashboard.

## Deep Dive

Creating a company is the first action you take in Paperclip. You can do it through the UI or the REST API. Here is how to create a company via the API:

```bash
curl -X POST http://localhost:3100/api/companies \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme AI Dev Shop",
    "description": "AI-powered software development company",
    "monthlyBudget": 50000
  }'
```

This creates a company named "Acme AI Dev Shop" with a monthly budget of 50000 cents ($500.00). The budget is specified in cents to avoid floating-point precision issues that plague dollar-based calculations.

The response includes the company's generated ID, which you will use in all subsequent API calls to manage this company's resources.

Companies have three statuses that control their lifecycle:

- **active**: The company is operational. Agents can run heartbeats, check out issues, and incur costs.
- **paused**: The company is temporarily suspended. Agents stop receiving heartbeats and cannot check out new issues, but existing data is preserved.
- **archived**: The company is permanently inactive. All operations cease. This is used when a company's work is complete.

You update a company's status (or any other attribute) with a PATCH request:

```bash
# Pause a company
curl -X PATCH http://localhost:3100/api/companies/{companyId} \
  -H "Content-Type: application/json" \
  -d '{ "status": "paused" }'

# Update the monthly budget
curl -X PATCH http://localhost:3100/api/companies/{companyId} \
  -H "Content-Type: application/json" \
  -d '{ "monthlyBudget": 100000 }'
```

The first command pauses the company, halting all agent activity. The second command doubles the monthly budget to $1,000.00 (100000 cents).

Paperclip supports running multiple companies simultaneously. Each company operates independently with its own org chart, budgets, and governance. This is useful for agencies, research labs, or organizations that want to separate concerns across different AI initiatives.

```bash
# List all companies
curl http://localhost:3100/api/companies
```

This returns an array of all companies in the Paperclip instance, letting you manage your portfolio of AI organizations from a single control plane.

## Common Pitfalls

1. **Setting budget in dollars instead of cents** — A budget of 500 means $5.00, not $500.00. Always multiply by 100 when converting from dollars.
2. **Forgetting that archived is permanent** — Once a company is archived, it cannot be reactivated. Use paused for temporary suspension.

## Best Practices

1. **Start with a conservative budget** — You can always increase the budget later. Starting low helps you understand cost patterns before committing.
2. **Use descriptions meaningfully** — The company description is surfaced to agents as context. A clear description helps agents understand the organization's purpose.

## Summary

- Company is the top-level unit in Paperclip, containing all other resources
- Created via POST /api/companies with name, description, and monthlyBudget (in cents)
- Three statuses: active (operational), paused (suspended), archived (permanent)
- Updated via PATCH /api/companies/{companyId}
- Multiple companies can run independently in a single Paperclip instance

## Code Examples

**Create a new company with a $500 monthly budget**

```bash
curl -X POST http://localhost:3100/api/companies \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme AI Dev Shop",
    "description": "AI-powered software development company",
    "monthlyBudget": 50000
  }'
```


## Resources

- [Paperclip Companies Documentation](https://paperclip.ing/docs) — Official documentation for managing companies in Paperclip

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*