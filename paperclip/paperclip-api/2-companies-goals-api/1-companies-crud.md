---
source_course: "paperclip-api"
source_lesson: "paperclip-api-companies-crud"
---

# Companies CRUD

## Introduction

Companies are the top-level organizational unit in Paperclip. Every agent, issue, goal, and cost event belongs to a company. Before you can do anything meaningful with the API, you need to understand how to create, read, update, and archive companies. This lesson covers the full companies CRUD lifecycle.

## Key Concepts

- **Company**: The top-level container for all resources in Paperclip. Agents, issues, goals, projects, and cost events are all scoped to a company.
- **Budget in Cents**: Monthly budgets are specified in cents, not dollars. A budget of 10000 means $100.00.
- **Company Status**: Companies can be `active`, `paused`, or `archived`. Archiving hides the company from default listings without deleting data.
- **Access Control**: Users and agents only see companies within their authorized scope.

## Real World Context

When you set up a new client project, you create a company in Paperclip to contain all the agents, issues, and cost tracking for that engagement. Setting the monthly budget in cents ensures precise financial controls. When the project ends, you archive the company rather than deleting it, preserving the audit trail for future reference.

## Deep Dive

### Listing Companies

Retrieve all companies accessible to the authenticated user or agent:

```bash
GET /api/companies
```

The response is a JSON array of company objects. Only companies within the caller's scope are returned.

```json
[
  {
    "id": "company-1",
    "name": "Acme Corp",
    "description": "Widget manufacturing",
    "budgetMonthlyCents": 100000,
    "status": "active"
  },
  {
    "id": "company-2",
    "name": "Beta Inc",
    "description": "Software consulting",
    "budgetMonthlyCents": 50000,
    "status": "active"
  }
]
```

Each company object includes its ID, name, description, monthly budget in cents, and current status.

### Retrieving a Single Company

Get the details of a specific company:

```bash
GET /api/companies/{companyId}
```

This returns the full company object including name, description, budget, and status. Returns 404 if the company does not exist or the caller lacks access.

### Creating a Company

Create a new company with a name and optional description:

```bash
POST /api/companies
```

The request body requires a name and optionally accepts a description:

```json
{
  "name": "Acme Corp",
  "description": "Widget manufacturing and distribution"
}
```

The API returns the created company object with a server-generated ID. The budget defaults to zero and status defaults to active.

### Updating a Company

Update one or more fields on an existing company:

```bash
PATCH /api/companies/{companyId}
```

The request body can include any combination of updatable fields:

```json
{
  "name": "Acme Corporation",
  "description": "Updated description",
  "budgetMonthlyCents": 200000,
  "status": "active"
}
```

Note that the budget is in cents. Setting `budgetMonthlyCents` to 200000 means $2,000.00 per month. PATCH is idempotent.

### Archiving a Company

To remove a company from default listings without deleting its data, set its status to `archived`:

```json
{
  "status": "archived"
}
```

Archived companies are hidden from the default `GET /api/companies` listing but their data (agents, issues, costs) remains intact. This is the preferred way to decommission a company.

## Common Pitfalls

- **Specifying budget in dollars instead of cents**: The API expects cents. A budget of 100 means $1.00, not $100.00. Always multiply your dollar amount by 100.
- **Deleting instead of archiving**: There is no delete endpoint for companies. Use the `archived` status to hide a company while preserving its data.
- **Forgetting that access is scoped**: An API key for Agent A in Company 1 cannot read Company 2's data.

## Best Practices

- **Always set a budget when creating a company** by following up with a PATCH to set `budgetMonthlyCents`.
- **Use descriptive company names** that make it easy to identify the company in logs and dashboards.
- **Archive companies instead of leaving them active** when work is complete.

## Summary

- Companies are the top-level organizational unit containing all Paperclip resources
- CRUD operations: GET (list/detail), POST (create), PATCH (update)
- Monthly budgets are specified in cents (10000 = $100.00)
- Status values: active, paused, archived
- Archiving hides a company from listings without deleting data
- Access control ensures users and agents only see their authorized companies

## Code Examples

**Creating a company and then setting its monthly budget via sequential API calls**

```bash
# Create a company
curl -X POST http://localhost:3100/api/companies \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Acme Corp", "description": "Widget manufacturing"}'

# Set the monthly budget to $2,000
curl -X PATCH http://localhost:3100/api/companies/company-1 \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"budgetMonthlyCents": 200000}'
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering the Companies API
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code with Companies API implementation

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*