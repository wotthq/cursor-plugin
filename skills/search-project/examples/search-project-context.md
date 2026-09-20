---
name: search-project-context-example
description: Example workflow for using wott project search to retrieve relevant notebook context for a task.
metadata:
  short-description: Find wott project context
---

# Search Project Context

## User request

> Before updating my deployment notebook, find the notes about our production deployment process.

## Workflow

Identify the project if necessary:

```text
get_projects
```

Search:

```text
search_project
```

with:

```json
{
  "project_id": "PROJECT_ID",
  "query": "production deployment process"
}
```

Inspect the returned notebook results.

If one notebook is clearly the intended target:

```text
get_notebook
```

Use the returned notebook content as context for the later update.

If multiple notebooks are plausible, ask the user which one they mean before calling:

```text
update_notebook
```

## Expected behavior

Search is only used for discovery.

The model must not modify the notebook until the user has explicitly requested the update and the target notebook is sufficiently identified.
