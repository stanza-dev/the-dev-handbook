---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-projects-workspaces"
---

# Projects & Workspaces

## Introduction

If goals are the "why," projects are the "what." Projects in Paperclip represent concrete deliverables — an API, a frontend app, a documentation site. Workspaces connect projects to the actual code repositories and development environments where agents do their work.

## Key Concepts

- **Project**: A concrete deliverable linked to one or more goals, containing issues that agents work on
- **Workspace**: A connection between a project and a code repository or development environment
- **Primary Workspace**: The main workspace where agents operate by default when working on a project's issues

## Real World Context

Your AI company has a goal to "Ship MVP by Q2." Under that goal, you create two projects: "Backend API" and "Frontend Dashboard." Each project gets a workspace pointing to its Git repository. When an agent checks out an issue from the Backend API project, it automatically gets the workspace context — the repo URL, branch, and environment — so it knows where to write code.

## Deep Dive

Projects sit between goals and issues in the Paperclip hierarchy. They group related issues into a coherent deliverable and link that deliverable to the strategic goals that justify it.

Creating a project is straightforward:

```bash
# Create a project (projects are created within the company context)
curl -X POST http://localhost:3100/api/companies/{companyId}/projects \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Backend API",
    "description": "REST API with authentication, CRUD endpoints, and webhook support"
  }'
```

This creates a project named "Backend API" within the specified company. The description helps agents understand the scope of the project when they are assigned issues from it.

Once you have a project, you connect it to a code repository by creating a workspace:

```bash
# Add a workspace to a project
curl -X POST http://localhost:3100/api/projects/{projectId}/workspaces \
  -H "Content-Type: application/json" \
  -d '{
    "name": "backend-repo",
    "repositoryUrl": "https://github.com/acme/backend-api",
    "branch": "main",
    "isPrimary": true
  }'
```

This creates a workspace that connects the Backend API project to its GitHub repository. The `isPrimary` flag marks this as the default workspace — when agents work on issues from this project, they operate in this workspace unless specified otherwise.

A project can have multiple workspaces. For example, a full-stack project might have one workspace for the backend repo and another for the frontend repo. But only one workspace can be primary.

The workspace is what makes agents productive. Without it, an agent knows what to do (the issue description) and why (the goal) but not where. The workspace provides the "where" — the repository, branch, and environment configuration that the agent needs to write and test code.

When an agent starts a heartbeat and checks out an issue, the full context it receives includes:

```bash
# Context chain surfaced to agent during execution:
# 1. Company: name, description
# 2. Goal: title, description, level
# 3. Project: name, description
# 4. Workspace: repository URL, branch
# 5. Issue: title, description, priority, parent issues
```

This complete context chain is what makes Paperclip agents effective. They are not just executing a task in isolation — they understand the full organizational context.

## Common Pitfalls

1. **Creating projects without workspaces** — A project without a workspace leaves agents with no development environment. Always add at least one workspace.
2. **Forgetting to set a primary workspace** — If no workspace is marked primary, agents may not know which environment to use by default.

## Best Practices

1. **One project per deliverable** — Do not lump multiple unrelated deliverables into a single project. Keep projects focused and cohesive.
2. **Name workspaces descriptively** — Use names like "backend-repo" or "docs-site" rather than "workspace-1" so the context is clear to agents.

## Summary

- Projects represent concrete deliverables (the "what") linked to goals (the "why")
- Workspaces connect projects to code repositories via POST /api/projects/{projectId}/workspaces
- The primary workspace is where agents operate by default
- Agents receive the full context chain: company → goal → project → workspace → issue
- A project can have multiple workspaces, but only one primary

## Code Examples

**Create a primary workspace connecting a project to a Git repository**

```bash
curl -X POST http://localhost:3100/api/projects/{projectId}/workspaces \
  -H "Content-Type: application/json" \
  -d '{
    "name": "backend-repo",
    "repositoryUrl": "https://github.com/acme/backend-api",
    "isPrimary": true
  }'
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering projects and workspaces

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*