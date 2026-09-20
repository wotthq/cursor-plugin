---
name: update-project-example
description: Example workflow for updating an existing project in wott.
metadata:
  short-description: Update an wott project
---

# Update Project

## User request

> Rename my robotics project to Physical AI Research.

## Workflow

1. Search for the robotics project.
2. Determine whether exactly one project matches.
3. If one project matches, use its project ID.
4. Call `update_project`.
5. Change only the project name.
6. Verify the returned project.
7. Report the successful update.

## MCP interaction

```text
search_project
    query: "robotics"
        ↓
One matching project
        ↓
update_project
    project_id: "project_123"
    name: "Physical AI Research"
        ↓
Successful result
        ↓
Report update
```

## Preserve unrelated information

If the existing project contains:

```text
name
description
status
created_at
updated_at
```

and the user only asks to rename it, do not overwrite the description or other unrelated fields.

Send only the requested update fields when the MCP schema allows partial updates.

## Ambiguous project

If search returns:

```text
Robotics Research
Robotics Hardware
Robotics Startup
```

do not choose one automatically.

Ask the user which project they mean.

## Success

Only report the rename after the MCP tool confirms success.

Example:

> Renamed "Robotics Research" to "Physical AI Research".
