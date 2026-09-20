---
name: update-notebook-example
description: Example workflow for updating wott notebook content, metadata, name, or location.
metadata:
  short-description: Update an wott notebook
---

# Update Notebook

## User request

> Rename my robotics notebook to Humanoid Manipulation Research.

## Workflow

1. Search for the robotics notebook.
2. Determine whether exactly one notebook matches.
3. If one notebook matches, use its notebook ID.
4. Call `update_notebook`.
5. Change only the notebook name.
6. Verify the returned notebook.
7. Report the successful update.

## MCP interaction

```text
search notebook
    query: "robotics"
        ↓
One matching notebook
        ↓
update_notebook
    notebook_id: "notebook_123"
    name: "Humanoid Manipulation Research"
        ↓
Successful result
        ↓
Report update
```

## Preserve unrelated information

If the notebook contains:

```text
name
description
project_id
content
created_at
updated_at
```

and the user only asks to rename it, do not overwrite the description, project relationship, or content.

Only modify the requested field when the MCP schema allows partial updates.

## Ambiguous notebook

If search returns multiple candidates:

```text
Robotics Research
Robotics Hardware
Robotics Manipulation
```

do not choose one arbitrarily.

Ask the user which notebook they mean.

## Success

Only report the rename after wott confirms that the update succeeded.

Example:

> Renamed the notebook to "Humanoid Manipulation Research".
