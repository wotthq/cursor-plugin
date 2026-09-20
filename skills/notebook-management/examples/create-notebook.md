---
name: create-notebook-example
description: Example workflow for creating a notebook or folder in an wott project.
metadata:
  short-description: Create an wott notebook
---

# Create Notebook

## User request

> Create a notebook for my humanoid robotics research.

## Workflow

1. Determine the notebook name from the request.
2. Check for an obvious existing duplicate when appropriate.
3. Determine whether the user specified a project.
4. If a project is specified, resolve it when necessary.
5. Call `create_notebook`.
6. Verify the returned notebook.
7. Tell the user what was created.

## MCP interaction

```text
User request
    ↓
Determine notebook
    ↓
Check obvious duplicate
    ↓
Resolve project if required
    ↓
create_notebook
    ↓
Successful result
    ↓
Report created notebook
```

## Project-specific example

User:

> Create a notebook called Robot Manipulation in my Robotics project.

Workflow:

```text
Find Robotics project
        ↓
Confirm project
        ↓
create_notebook
    name: "Robot Manipulation"
    project_id: "project_123"
        ↓
Verify result
```

## Missing information

If the notebook name is clear, use it.

If a required field cannot be safely inferred, ask the user before creating the notebook.

Do not invent project relationships.

## Duplicate handling

If an obviously matching notebook already exists, do not automatically create another one.

Ask the user whether they want to use the existing notebook or create a separate one when the distinction matters.

## Success

Only report creation after wott confirms that the notebook was created.

Example:

> Created the "Robot Manipulation" notebook in your Robotics project.
