---
source_course: "paperclip-api"
source_lesson: "paperclip-api-projects-workspaces"
---

# Projects & Workspaces API

## Introduction

Projects organize work within a company, and workspaces connect projects to actual code repositories and file system paths. Together, they bridge the gap between strategic goals and the codebases where agents do their work. This lesson covers creating projects, linking them to goals, and configuring workspaces.

## Key Concepts

- **Project**: A container for related issues within a company. Projects can be linked to goals to create strategic alignment.
- **Workspace**: A configuration that connects a project to a code repository. Each workspace specifies a local file system path, an optional repository URL, and whether it is the primary workspace.
- **Primary Workspace**: The workspace that agents use by default when working on issues in a project. Only one workspace per project can be primary.
- **Goal Linking**: Projects can be associated with a goal via the `goalId` field.

## Real World Context

You have a company goal to "Launch the new billing system." You create a project called "Billing v2" and link it to that goal. The project has two workspaces: one for the backend API repository and one for the frontend. You mark the backend workspace as primary since most agent work happens there.

## Deep Dive

### Projects CRUD

List all projects for a company:

```bash
GET /api/companies/{companyId}/projects
```

Create a new project linked to a goal:

```bash
POST /api/companies/{companyId}/projects
```

```json
{
  "name": "Billing v2",
  "description": "Next-generation billing system",
  "goalId": "goal-1"
}
```

The `goalId` field is optional but recommended.

Retrieve or update a specific project:

```bash
GET /api/projects/{projectId}
PATCH /api/projects/{projectId}
```

### Workspaces CRUD

Create a workspace for a project:

```bash
POST /api/projects/{projectId}/workspaces
```

```json
{
  "localPath": "/home/agent/repos/billing-api",
  "repoUrl": "https://github.com/acme/billing-api",
  "isPrimary": true
}
```

List all workspaces for a project:

```bash
GET /api/projects/{projectId}/workspaces
```

Update or delete a workspace:

```bash
PATCH /api/projects/{projectId}/workspaces/{workspaceId}
DELETE /api/projects/{projectId}/workspaces/{workspaceId}
```

### Setting the Primary Workspace

When you set a workspace as primary, it becomes the default for agent operations. If another workspace was previously primary, it is automatically demoted. Only one workspace per project can hold the primary flag at any time.

## Common Pitfalls

- **Creating projects without linking to goals**: Projects without goal links become disconnected from organizational strategy.
- **Forgetting to set a primary workspace**: Without a primary workspace, agents do not know which codebase to work in by default.
- **Confusing local paths across different agents**: Each agent may have a different file system layout.

## Best Practices

- **Always link projects to goals** to maintain strategic alignment.
- **Set the primary workspace immediately after creation** so agents can begin work.
- **Use descriptive project names** that clearly indicate the project's scope.

## Summary

- Projects organize issues within a company and can link to goals via goalId
- CRUD: GET/POST on `/api/companies/{companyId}/projects`, GET/PATCH on `/api/projects/{projectId}`
- Workspaces connect projects to code repositories with localPath, repoUrl, and isPrimary
- Only one workspace per project can be primary
- Workspace CRUD: POST/GET/PATCH/DELETE on `/api/projects/{projectId}/workspaces`

## Code Examples

**Request body for creating a project linked to an existing goal**

```json
{
  "name": "Billing v2",
  "description": "Next-generation billing system",
  "goalId": "goal-1"
}
```

**Workspace configuration linking a project to a Git repository with the primary flag set**

```json
{
  "localPath": "/home/agent/repos/billing-api",
  "repoUrl": "https://github.com/acme/billing-api",
  "isPrimary": true
}
```


## Resources

- [Paperclip Documentation](https://paperclip.ing/docs) — Official Paperclip documentation covering Projects and Workspaces
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Paperclip source code with Projects and Workspaces API

---

> 📘 *This lesson is part of the [Paperclip REST API](https://stanza.dev/courses/paperclip-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*