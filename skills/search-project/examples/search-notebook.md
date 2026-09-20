---
name: search-notebook-example
description: Example workflow for searching notebooks within a specific wott project.
metadata:
  short-description: Search wott notebooks
---

# Search Notebook Knowledge

## User request

> Search my AI Research project for humanoid robotics.

## Workflow

First identify the project if the project ID is not already known.

```text
get_projects
```

Find:

```text
AI Research
```

Then search:

```text
search_project
```

with:

```json
{
  "project_id": "PROJECT_ID",
  "query": "humanoid robotics"
}
```

## Expected behavior

The model should return the most relevant notebook results.

Example:

```text
I found 3 relevant notebooks:

1. Humanoid Robotics.md
   Contains research on humanoid robot systems and locomotion.

2. Robotics Simulation.md
   Contains simulation and testing notes.

3. Robot Learning.md
   Contains related robotics research.

I can open any of these notebooks if you want.
```

Do not modify any notebook.
