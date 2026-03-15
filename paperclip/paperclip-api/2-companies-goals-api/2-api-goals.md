---
source_course: "paperclip-api"
source_lesson: "paperclip-api-goals"
---

# Goals API

## Introduction

Goals give your Paperclip organization direction. They cascade from the company level down to teams and individual agents, creating alignment across the entire hierarchy. The Goals API lets you create, read, and update goals at any level, linking strategic objectives to the work agents actually perform.

## Key Concepts

- **Goal Levels**: Goals exist at three levels: `company`, `team`, and `agent`. Company goals define strategic direction, team goals break those into departmental objectives, and agent goals specify individual deliverables.
- **Goal Attributes**: Each goal has a title, description, level, and status.
- **Cascading Alignment**: Goals at lower levels can reference higher-level goals, creating a hierarchy from company strategy down to individual agent tasks.
- **Goal Status**: Tracks whether a goal is active, completed, or in another lifecycle state.

## Real World Context

A company sets a goal to "Reduce infrastructure costs by 20%." The DevOps team creates a team-level goal to "Migrate to spot instances." Individual agents get goals like "Audit current EC2 usage" and "Write Terraform migration scripts." The Goals API makes this hierarchy explicit and queryable.

## Deep Dive

### Listing Goals

Retrieve all goals for a company:

```bash
GET /api/companies/{companyId}/goals
```

This returns an array of goal objects:

```json
[
  {
    "id": "goal-1",
    "title": "Reduce infrastructure costs by 20%",
    "description": "Optimize cloud spending across all services",
    "level": "company",
    "status": "active"
  },
  {
    "id": "goal-2",
    "title": "Migrate to spot instances",
    "description": "Move non-critical workloads to spot instances",
    "level": "team",
    "status": "active"
  }
]
```

### Creating a Goal

Create a new goal for a company:

```bash
POST /api/companies/{companyId}/goals
```

```json
{
  "title": "Improve test coverage to 90%",
  "description": "Increase unit and integration test coverage across all services",
  "level": "team",
  "status": "active"
}
```

The `level` field must be one of `company`, `team`, or `agent`.

### Retrieving and Updating a Goal

Get a specific goal by ID:

```bash
GET /api/goals/{goalId}
```

Update a goal's attributes:

```bash
PATCH /api/goals/{goalId}
```

```json
{
  "status": "completed",
  "description": "Achieved 92% coverage across all services"
}
```

### Cascading Alignment

Goals at different levels connect to form a hierarchy. When you create a project, you can link it to a goal. When you create issues within that project, they inherit the goal context. This creates a clear chain: company goal to team goal to agent goal to project to issue.

## Common Pitfalls

- **Creating goals without setting the level**: The `level` field is required. Omitting it causes a 422 validation error.
- **Not linking goals to projects and issues**: Goals without linked work items become orphaned objectives.
- **Confusing goal levels with permissions**: Goal levels describe organizational scope, not access control.

## Best Practices

- **Start with company-level goals** and cascade downward to ensure alignment.
- **Keep goal titles concise and measurable**: "Reduce costs by 20%" is better than "Work on cost optimization."
- **Update goal status when work is complete** to maintain an accurate picture of progress.

## Summary

- Goals exist at three levels: company, team, and agent
- CRUD operations: GET (list/detail), POST (create), PATCH (update)
- Each goal has title, description, level, and status
- Goals cascade from company strategy down to individual agent tasks
- Link goals to projects and issues to create traceable alignment

## Code Examples

**Request body for creating a company-level goal with a measurable title**

```json
{
  "title": "Reduce infrastructure costs by 20%",
  "description": "Optimize cloud spending across all services",
  "level": "company",
  "status": "active"
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering the Goals API

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*