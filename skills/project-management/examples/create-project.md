---
name: create-project-example
description: Example workflow for creating a new project in wott.
metadata:
  short-description: Create an wott project
---

# Create Project

## User request

> Create a project for my humanoid robotics research.

## Workflow

1. Determine the project name from the request.
2. Check whether an obvious matching project already exists.
3. If no duplicate is found, call `create_project`.
4. Use the returned project as the source of truth.
5. Tell the user that the project was created.

## MCP interaction

```text
search_project
    query: "humanoid robotics"
        ↓
No obvious duplicate
        ↓
create_project
    name: "Humanoid Robotics"
    description: "Research on humanoid robotics"
        ↓
Successful result
        ↓
Report created project
```

## Important behavior

Do not create a project when the user is only discussing an idea.

For example:

> I'm thinking about starting a robotics research project.

This does not necessarily mean the user wants a project created.

Instead, clarify if necessary.

## Missing information

If the project name can be safely inferred, use it.

If a required field cannot be inferred safely, ask the user before creating the project.

Do not invent important project information.

## Success

Only report creation after the MCP tool confirms success.

Example:

> Created the "Humanoid Robotics" project in wott.
